---
title: Golang - slice 和 map 的正确打开方式
tags:
  - slice
  - map
categories:
  - golang
date: 2016-12-25 15:47:37
---

# 提出问题

1. 对一个slice的操作什么时候会影响别的slice，什么时候不会？
2. 遍历slice的时候需要注意什么？
3. slice元素的类型该不该用指针？
4. 遍历map的时候需要注意什么？
5. map的value类型该不该用指针？
6. 为什么slice的元素可以取地址，而map的value却不可以？

我们将先分析一些代码片段，再对这些问题作出回答。

# slice代码片段分析
在Golang中，slice是array的描述符，对slice的操作实际上是对它指向的array的操作。

一个slice由三部分构成: ptr(指向array的指针), len(当前大小), cap(最大容量)。

![](/static/golang_slice_and_map/slice_1.png)
![](/static/golang_slice_and_map/slice_2.png)

-----------------
```go
a := [4]int{1, 2, 3, 4}
fmt.Printf("a: %v\n", a)
s1 := a[:2]
s1 = append(s1, 9)
fmt.Printf("s1 = a[:2]\n")
fmt.Printf("s1: %v\n", s1)
fmt.Printf("s1.len: %v\n", len(s1))
fmt.Printf("s1.cap: %v\n", cap(s1))
s2 := a[1:]
fmt.Printf("s2 = a[1:]\n")
fmt.Printf("s2: %v\n", s2)
fmt.Printf("s2.len: %d\n", len(s2))
fmt.Printf("s2.cap: %d\n", cap(s2))
s1[1] = 0
fmt.Printf("s1[1] = 0\n")
fmt.Printf("a: %v\n", a)
fmt.Printf("s1: %v\n", s1)
fmt.Printf("s2: %v\n", s2)
fmt.Printf("&a[1]: %p\n", &a[1])
fmt.Printf("&s1[1]: %p\n", &s1[1])
fmt.Printf("&s2[0]: %p\n", &s2[0])
```

输出：
```bash
a: [1 2 3 4]
s1 = a[:2]
s1: [1 2]
s1.len: 2
s1.cap: 4
s2 = a[1:]
s2: [2 3 4]
s2.len: 3
s2.cap: 3
s1[1] = 0
a: [1 0 3 4]
s1: [1 0]
s2: [0 3 4]
&a[1]: 0xc420012288
&s1[1]: 0xc420012288
&s2[0]: 0xc420012288
```
对一个array进行slicing操作得到的slice指向该array。

对一个slice进行slicing得到新的slice与原slice指向同一个array。

此时这个两个slice对它们的重叠部分的元素进行写操作是会互相影响的，因为它们操作的是同一个array。

-----------------

```go
s1 := []int{1, 2, 3, 4}
fmt.Printf("s1: %v\n", s1)
fmt.Printf("&s1[0]: %p\n", &s1[0])
fmt.Printf("s1.len: %d\n", len(s1))
fmt.Printf("s1.cap: %d\n", cap(s1))

s2 := s1
fmt.Printf("&s2[0]: %p\n", &s2[0])
fmt.Printf("s2: %v\n", s1)
fmt.Printf("s2.len: %d\n", len(s2))
fmt.Printf("s2.cap: %d\n", cap(s2))

s2 = append(s2, 9)
fmt.Printf("s2 = append(s2, 9)\n")
fmt.Printf("s1: %v\n", s1)
fmt.Printf("s2: %v\n", s2)
fmt.Printf("&s1[0]: %p\n", &s1[0])
fmt.Printf("&s2[0]: %p\n", &s2[0])
fmt.Printf("s2.len: %d\n", len(s2))
fmt.Printf("s2.cap: %d\n", cap(s2))
```

输出：
```bash
s1: [1 2 3 4]
&s1[0]: 0xc420012280
s1.len: 4
s1.cap: 4
&s2[0]: 0xc420012280
s2: [1 2 3 4]
s2.len: 4
s2.cap: 4
s2 = append(s2, 9)
s1: [1 2 3 4]
s2: [1 2 3 4 9]
&s1[0]: 0xc420012280
&s2[0]: 0xc42000e200
s2.len: 5
s2.cap: 8
```
当一个slice扩容的时候，底层会新建一个大小为该slice的容量的2倍的array，此后s1和s2指向不同的array，操作也就不会互相影响了。

-----------------

```go
s := []int{1, 2, 3, 4}
fmt.Printf("s: %v\n", s)

for i, v := range s {
    fmt.Printf("v: %d\n", v)
    fmt.Printf("s[%d]: %d\n", i, s[i])
    fmt.Printf("&v: %p\n", &v)
    fmt.Printf("&s[%d]: %p\n", i, &s[i])
}
```
输出：
```bash
s: [1 2 3 4]
v: 1
s[0]: 1
&v: 0xc42000a328
&s[0]: 0xc420012280
v: 2
s[1]: 2
&v: 0xc42000a328
&s[1]: 0xc420012288
v: 3
s[2]: 3
&v: 0xc42000a328
&s[2]: 0xc420012290
v: 4
s[3]: 4
&v: 0xc42000a328
&s[3]: 0xc420012298
```
遍历slice时，变量v只是对当前遍历元素的一个拷贝，修改其值或取其指针都是没有意义的。

# map代码片段分析

Golang的map是用hash table实现的，这个hash table由一些bucket组成，每个bucket能装8对key/value。

key/value中key的hash值的低位决定它们在哪个bucket，高位是它们在bucket中的唯一标识。

![](/static/golang_slice_and_map/map_1.png)

------------

```go
m := make(map[int]int)
m[0] = 10
m[1] = 11
m[2] = 12

fmt.Printf("First Range:\n")
for k, v := range m {
    fmt.Printf("k: %d, v: %d\n", k, v)
}

fmt.Printf("Second Range:\n")
for k, v := range m {
    fmt.Printf("k: %d, v: %d\n", k, v)
}
```

输出：
```bash
First Range:
k: 2, v: 12
k: 0, v: 10
k: 1, v: 11
Second Range:
k: 0, v: 10
k: 1, v: 11
k: 2, v: 12
```
遍历map的顺序是随机的。

-----------------

```go
m := make(map[int]int)
m[0] = 10
fmt.Printf("&m[0]: %p\n", &m[0])
```

输出：
```bash
cannot take the address of m[0]
```
无法获取value的地址

-----------------
```bash
type People struct {
    Name string
    Age  int
}
m := make(map[string]People)
m["Li"] = People{Name: "Li", Age: 10}
// m["Li"].Age = 20 // cannot assign to struct field m["Li"].Age in map

p, _ := m["Li"]
p.Age = 20
fmt.Printf("m[\"Li\"]: %+v\n", m["Li"])

m["Li"] = People{Name: "Li", Age: 20}
fmt.Printf("m[\"Li\"]: %+v\n", m["Li"])
```
输出：
```bash
m["Li"]: {Name:Li Age:10}
m["Li"]: {Name:Li Age:20}
```
对p的修改只是对一份"Li"所指向的Peolple的拷贝的修改。

没有办法对map中key所对应的某个value进行修改，只能通过赋值一个新的value来代替。

# 问题解答
1. 对一个slice的操作什么时候会影响别的slice，什么时候不会？

>　当两个slice指向同一个array，且它们有重叠的元素时，对重叠元素的进行写操作会互相影响，其他情况不会。

2. 遍历slice的时候需要注意什么？

>　遍历时的变量v只是元素的一份拷贝。修改元素的值要使用s[i]，取元素地址要用&s[i]。

3. slice元素的类型该不该用指针？

>　在元素的类型为非指针的情况下，对一个slice的某个元素进行取指针保存，等该slice扩容后，对该指针的操作只会影响旧slice，不会影响新slice。如果元素的类型本身是指针类型，就不用进行取指针，直接保存指针就好了，后续对该指针的操作会影响到的新的slice。

>　在元素为非指针类型,且元素是struct类型的情况下，后续的操作会增加很多拷贝消耗，例如给slice元素赋值和遍历slice。

>　因此，如果需要对元素进行取指针保存以方便后续操作，或者元素的类型是struct类型，建议使用指针。

4. 遍历map的时候需要注意什么？

>　遍历时的变量v只是value的一份拷贝。

>　遍历的顺序是随机的。这是Golang故意这样实现的。map保存key/value的方式与key的插入顺序无关，同时map也不另外保存key的插入顺序，因此遍历的顺序完全由当前map内部key的hash情况决定的。如果按照非随机算法来遍历map，Golang的确可以做到在不扩容且不增减key的情况下，多次遍历的顺序一致。但是Golang想告诉开发者它无法对遍历key的遍历顺序作出保证，因此干脆就在遍历算法中加入伪随机数，来让每次遍历顺序尽量不同。

5. map的value类型该不该用指针？

>　在value的类型为非指针类型，且为struct类型的情况下，要想修改某个key对应的value的某个属性，只能通过赋值一个新的struct，无法直接修改该value的某个属性。

>　在value为非指针类型,且元素是struct类型的情况下，后续的操作会增加很多拷贝消耗，例如给map赋值和遍历map。

>　因此，如果value为struct类型，建议使用指针。

6. 为什么slice的元素可以取地址，而map的value却不可以？

>　slice是基于array实现的，且array是暴露给开发者的（例如对array进行slicing操作获得slice）。array可以取元素地址，slice很自然也可以。

>　map的内部结构没有直接暴露给开发者的。当map进行扩容时，会将原有的key/value重新hash并拷贝到新的空间，Golang为了避免开发者对旧空间的访问，因此不给开发者任何获取value地址的机会。


# 参考资料

1. https://blog.golang.org/go-slices-usage-and-internals
2. https://blog.golang.org/go-maps-in-action
3. https://www.goinggo.net/2013/12/macro-view-of-map-internals-in-go.html
