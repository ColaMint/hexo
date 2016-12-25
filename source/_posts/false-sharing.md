---
title: False Sharing
date: 2016-05-21 10:49:07
categories:
  - linux
tags:
  - false sharing
---
# Cache Line
CPU中的缓存被划分为大小一致的缓存行，通常缓存行的大小是32Bytes或64Bytes。CPU以缓存行为单位把热数据从内存加载到缓存中。

# False Sharing
在多核时代，当程序中运行在不同CPU核的线程修改处于同一缓存行的变量时，这时处理器需要保证不同CPU核的缓存的一致性而做一些额外的工作，从而损耗程序的性能。在这种现象中，不同线程不是因为修改同一变量，而是因为修改同一缓存行中的变量而发生冲突，所以这种现象被称为`False Sharing`。

![False Sharing](/static/false-sharing/False_Sharing.gif)

# MESI Protocol
为了确保高速缓存一致性，多处理器一般采用MESI(Modified/Exclusive/Shared/Invalid)协议。

* 当一个缓存行第一次被加载进缓存时，它被标记为`Exclusive`状态。只要缓存行的状态为`Exclusive`，对该缓存行的访问都是直接在该缓存行上进行。

* 当被标记为`Exclusive`状态的缓存被另一个CPU加载到缓存中时，它们的状态被标记为`Shared`。

* 当处于`Shared`状态的缓存行被修改时，它被标记为`Modified`状态，其它缓存中该缓存行被标记为`Invalid`状态。

* 当处于`Modified`状态的缓存行被其他CPU访问时，该缓存行的数据被刷回内存，并标记状态为`Shared`。访问该缓存行的其他CPU触发一个高速缓存缺失，从内存将该缓存行加载到Cache中，并标记为`Shared`状态。

# Avoid False Sharing
避免False Sharing主要有两种方法：`填充数组元素间隙`和`线程本地变量拷贝`

## 填充数组元素间隙
```c
struct ThreadParams
{
    // For the following 4 variables: 4*4 = 16 bytes
    unsigned long thread_id;
    unsigned long v; // Frequent read/write access variable
    unsigned long start;
    unsigned long end;

    // expand to 64 bytes to avoid false-sharing
    // (4 unsigned long variables + 12 padding)*4 = 64
    int padding[12];
};
__declspec (align(64)) struct ThreadParams Array[10];
```

假设缓存行的大小为64Bytes，`__declspec (align(64))`确保了数组的基址与缓存行的边界对齐。

在结构体中，`int padding[12];`对结构体大小进行填充，使其刚好为一个缓存行的大小。

如果无法确保数组的基址与缓存行的边界对其，那么就将结构的大小填充至缓存行大小的2倍。

如果数组是动态分配的，那么可以先申请较大的内存空间，再把数组的指针指向第一个与缓存行边界对其的位置。（注意：free内存时要对分配内存时得到的地址进行free，而不是对数组的基址进行free）


## 线程本地变量拷贝
```c
struct ThreadParams
{
    // For the following 4 variables: 4*4 = 16 bytes
    unsigned long thread_id;
    unsigned long v; //Frequent read/write access variable
    unsigned long start;
    unsigned long end;
};

void threadFunc(void *parameter)
{
    ThreadParams *p = (ThreadParams*) parameter;
    // local copy for read/write access variable
    unsigned long local_v = p->v;

    for(local_v = p->start; local_v < p->end; local_v++)
    {
    // Functional computation
    }

    p->v = local_v; // Update shared data structure only once
}
```
在线程中创建了对结构体中读写频繁的成员`v`的一个拷贝`local_v`，在后续操作中，线程只对结构体中的`thread_id`、`start`、`end`进行读操作，对本地变量`local_v`进行读写操作。直到所有工作都做完了，才把本地变量`local_v`的值写回结构体的`v`。这种方法只在最后一步才有可能发生`False Sharing`。

# 参考资料
 1. [Avoiding and Identifying False Sharing Among Threads](https://software.intel.com/en-us/articles/avoiding-and-identifying-false-sharing-among-threads)
