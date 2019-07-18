---
layout:     post
title:      free中的buffer和cache
date:       2019-07-18
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: Linux
tags:
    - buffer
    - cache
---

free中的buffer和cache:
redhat对free输出的解读

两者都是RAM中的数据。简单来说，buffer是即将要被写入磁盘的，而cache是被从磁盘中读出来的。 (free中的buffer和cach它们都是占用内存的)

A buffer is something that has yet to be "written" to disk.
A cache is something that has been "read" from the disk and stored for later use.
buffer
buffer : 作为buffer cache的内存，是块设备的写缓冲区。buffer是根据磁盘的读写设计的，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而提高系统性能。linux有一个守护进程定期清空缓冲内容（即写如磁盘），也可以通过sync命令手动清空缓冲。buffer是由各种进程分配的，被用在如输入队列等方面，一个简单的例子如某个进程要求有多个字段读入，在所有字段被读入完整之前，进程把先前读入的字段放在buffer中保存。

cache
cache: 作为page cache的内存, 文件系统的cache。cache经常被用在磁盘的I/O请求上，如果有多个进程都要访问某个文件，于是该文件便被做成cache以方便下次被访问，这样可提供系统性能。cache是把读取过的数据保存起来，重新读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除。　　如果 cache 的值很大，说明cache住的文件数很多。如果频繁访问到的文件都能被cache住，那么磁盘的读IO bi会非常小。

el6:
free 命令，在el6 el7上的输出是不一样的；
对于el6 ，看真正的有多少内存是free的，应该看 free的第二行！！！
```
[root@Linux ~]# free
             total       used       free     shared    buffers    cached
Mem:       8054344    1834624    6219720          0      60528    369948
-/+ buffers/cache:    1404148    6650196
Swap:       524280        144     524136
```
第1行:
total 内存总数: 8054344
used 已经使用的内存数: 1834624
free 空闲的内存数: 6219720
shared 当前已经废弃不用，总是0
buffers Buffer Cache内存数: 60528 （缓存文件描述信息）
cached Page Cache内存数: 369948 （缓存文件内容信息）

关系：total = used + free

第2行：
-/+ buffers/cache的意思相当于：
-buffers/cache 的内存数：1404148 (等于第1行的 used - buffers - cached)
+buffers/cache 的内存数：6650196 (等于第1行的 free + buffers + cached)

可见-buffers/cache反映的是被程序实实在在吃掉的内存，而+buffers/cache反映的是可以使用的内存总数。

释放掉被系统cache占用的数据:
手动执行sync命令（描述：sync 命令运行 sync 子例程。如果必须停止系统，则运行sync 命令以确保文件系统的完整性。sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件）

有关/proc/sys/vm/drop_caches的用法在下面进行了说明:

/proc/sys/vm/drop_caches (since Linux 2.6.16)
Writing to this file causes the kernel to drop clean caches,dentries and inodes from memory, causing that memory to become free.
To free pagecache, use echo 1 > /proc/sys/vm/drop_caches;
to free dentries and inodes, use echo 2 > /proc/sys/vm/drop_caches;
to free pagecache, dentries and inodes, use echo 3 > /proc/sys/vm/drop_caches.
Because this is a non-destructive operation and dirty objects are not freeable, the user should run sync first.

echo 3>/proc/sys/vm/drop_caches