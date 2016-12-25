---
title: 磁盘工具
date: 2016-07-30 08:13:58
categories:
  - linux
tags:
  - disk
---
# df - 查看磁盘空间占用情况
```bash
[root@everley]# df
Filesystem    512-blocks      Used Available Capacity  iused    ifree %iused  Mounted on
/dev/disk1     487849984 279458768 207879216    58% 34996344 25984902   57%   /
devfs                654       654         0   100%     1132        0  100%   /dev
map -hosts             0         0         0   100%        0        0  100%   /net
map auto_home          0         0         0   100%        0        0  100%   /home
```

# lsblk - 查看块设备信息
```bash
# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465,8G  0 disk
├─sda1   8:1    0   100M  0 part
├─sda2   8:2    0    80G  0 part
├─sda3   8:3    0 297,9G  0 part
├─sda4   8:4    0     1K  0 part
├─sda5   8:5    0    28G  0 part /
├─sda6   8:6    0   3,7G  0 part [SWAP]
└─sda7   8:7    0  56,2G  0 part /home
sr0     11:0    1  1024M  0 rom
```

# blkid - 查看分区ID和文件系统类型
```bash
# blkid
/dev/vda1: UUID="7fa9c421-0054-4555-b0ca-b470a97a3d84" TYPE="ext4"
/dev/vda2: UUID="7IvYzk-TnnK-oPjf-ipdD-cofz-DXaJ-gPdgBW" TYPE="LVM2_member"
/dev/mapper/vg_kvm-lv_root: UUID="a07b967c-71a0-4925-ab02-aebcad2ae824" TYPE="ext4"
/dev/mapper/vg_kvm-lv_swap: UUID="d7ef54ca-9c41-4de4-ac1b-4193b0c1ddb6" TYPE="swap"
```

# du - 查看目录下文件的总大小
```bash
# du -sh /home
3.9G    /home
```

# iostat 查看设备和分区的负载情况
```bash
# iostat
Linux 3.13.0-65-generic (everley)   07/30/2016  _x86_64_    (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.96    0.26    1.78    1.33    0.08   95.58

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
xvda             36.47       461.34        41.30 5374011333  481139068
```

# iotop 查看进程的磁盘读写速率
```bash
# iotop
Total DISK READ:       0.00 B/s | Total DISK WRITE:     106.14 K/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
  335 be/3 root        0.00 B/s   98.56 K/s  0.00 %  2.03 % [jbd2/sda6-8]
 4096 be/4 www-data    0.00 B/s    0.00 B/s  0.00 %  0.00 % apache2 -k start
    1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % init
    2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]
    3 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/0]
...
```

# cat /proc/[pid]/io 查看进程磁盘读写记录
```bash
# cat /proc/1275/io
rchar: 1661777
wchar: 7431
syscr: 1240
syscw: 123
read_bytes: 7335936
write_bytes: 12288
cancelled_write_bytes: 0
```

# mount/unmount 挂载设备
```bash
# mkdir /mnt/usb
# mount /dev/sdb1 /mnt/usb

#unmount /mnt/usb
```

# fdisk/gdisk/mkfs/mkswap - 磁盘分区、格式化工具
### [传送门](http://teaching.idallen.com/cst8207/13w/notes/720_partitions_and_file_systems.html])
