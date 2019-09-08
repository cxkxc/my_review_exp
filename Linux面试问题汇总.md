[TOC]

## Linux面试问题汇总

#### Linux的I/O模型介绍以及同步异步阻塞非阻塞的区别（超级重要）

> 阻塞IO、非阻塞IO、IO多路复用、信号驱动IO、异步IO



#### 文件系统的理解（EXT4，XFS，BTRFS）



#### 文件处理grep,awk,sed这三个命令必知必会

> [https://www.cnblogs.com/along21/p/10366886.html]:
>
> 1. grep：
>
> Linux系统中grep命令是一种强大的文本搜索工具，它能使用**正则表达式**搜索文本，并把匹配的行打印出来（匹配到的标红）。
>
> 2. sed：
>
> sed 是一种流编辑器，它一次处理一**行**内容。处理时，把当前处理的行存储在临时缓冲区中，称为“**模式空间**”（patternspace ），接着用sed 命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。然后读入下行，执行下一个循环。
>
> 3. awk
>
> awk是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它**支持用户自定义函数**和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。



#### IO复用的三种方法（select,poll,epoll）深入理解，包括三者区别，内部原理实现？

> https://blog.csdn.net/C1029323236/article/details/90551959
>
> 1. select：缺点：
>    - 单个进程能够监视的文件描述符的数量存在最大限制，通常是1024。由于select采用轮询的方式扫描文件描述符，文件描述符数量越多，性能越差；
>    - 内核/用户空间内存拷贝，select需要大量的句柄数据结构，产生巨大开销；
>    - select返回的是含有整个句柄的数组，应用程序需要遍历整个数组才能发现哪些句柄发生事件；
>    - select的触发方式为水平触发，应用程序如果没有完成对一个已经就绪的文件描述符进行IO操作，那么下次select调用还会将这些文件描述符通知进程
> 2. poll:
>    - 使用链表保存文件描述符，没有了文件描述符的限制，但其他的三个缺点依然存在
> 3. epoll:
>
> 1) 上面所说的select缺点都不存在，epoll使用了一个文件描述符管理了多个文件描述符。
> 2) epoll的设计与select完全不同，epoll通过在Linux内核申请一个简易的文件系统（文件一般使用什么数据结构实现？B+树），把原先的select/epoll调用分为3个部分：
>
> - 调用epoll_create()建立了一个epoll对象（在epoll文件系统中为这个句柄对象分配资源）
> - 调用epoll_ctl()向epoll对象添加这100万个连接的套接字（ip地址+端口号）
> - 调用epoll_wait()收集发生事件的连接
>
> 3) 每一个epoll对象都有一个独立的eventpoll结构体，用于存放epoll_ctl()方法向epoll对象中添加进来的事件，这些事件都会存放在这颗红黑树中，红黑树的增删查改的效率为logn，其中n为数的高度。
> 而所有添加到epoll对象中的事件都会与设备（网卡）驱动程序建立回调关系，当相应的事件发生时会调用这个回调方法，这个回调方法在内核中叫做ep_poll_back，它会将发生的事件添加到rdlist双链表中。
>
> 当调用epoll_wait()检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可，如果rdlist不为空，则把发生的事件复制到用户态，同时将事件数量返回给用户。
> **从上面的讲解可知：通过红黑树和双链表数据结构并结合回调机制，才有如此高效的epoll**
> epoll在使用ET模式的时候一定是非阻塞的，如果是阻塞的话，一个任务监控多个文件描述符时，当一个描述符阻塞就会使其他的描述符所在的任务失败（饿死）
>
> 4. 总结
>
> select和poll采用的都是轮询的方式，所以效率是O（n），而对于epoll采用的方式是回调的方式，说的就是当有就绪的文件描述符的时候，这个时候就会去触发回函数，回调函数就会将文件描述符对应的事件插入到内核就绪事件队列，内核最后在适当的时机将该就绪事件队列中的内容拷贝到用户空间。所以epoll_wait的时间复杂度为O（1）.epoll_wait有个缺点就是当活动连接多的时候，这个时候回调函数调用会被多次调用，这样效率也会下降，所以相对来说，epoll_wait适用于连接数多，但是活动连接少的情况下。
>
> 　　另外，采取的访问方式上来说也有区别，select和poll需要多次的在用户空间和内核空间上进行交互，而对于epoll来说，使用了mmap的方式，使得内核空间和用户控件映射到一个文件，这样更加高效。
>
> ![![enter description here][1]](https://img-blog.csdn.net/20170328134602071?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjY3Njg3NDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> ![![enter description here][2]](https://img-blog.csdn.net/20170328134632720?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjY3Njg3NDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> ![![enter description here][3]](https://img-blog.csdn.net/20170328134733346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjY3Njg3NDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> ![![enter description here][4]](https://img-blog.csdn.net/20170328134827088?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjY3Njg3NDE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> 



#### Epoll的ET模式和LT模式（ET的非阻塞）



#### 查询进程占用CPU的命令（注意要了解到used，buf，cache代表意义）

> **top !!!!**



#### linux的其他常见命令（kill，find，cp等等）



#### shell脚本用法



#### 硬连接和软连接的区别

> - **硬连接指通过索引节点来进行连接。在Linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。****在Linux中，多个文件名指向同一索引节点是存在的。比如：A是B的硬链接（A和B都是文件名），则A的目录项中的inode节点号与B的目录项中的inode节点号相同，即一个inode节点对应两个不同的文件名，两个文件名指向同一个文件，A和B对文件系统来说是完全平等的。删除其中任何一个都不会影响另外一个的访问。**
> - **另外一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有类似于Windows的快捷方式。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。**比如：A是B的软链接（A和B都是文件名），A的目录项中的inode节点号与B的目录项中的inode节点号不相同，A和B指向的是两个不同的inode，继而指向两块不同的数据块。但是A的数据块中存放的只是B的路径名（可以根据这个找到B的目录项）。A和B之间是“主从”关系，如果B被删除了，A仍然存在（因为两个是不同的文件），但指向的是一个无效的链接。
> - 依此您可以做一些相关的测试，可以得到以下全部结论：
>
> 0) 写入f1，f1、f2均改变
>
> 1).删除符号连接f3,对f1,f2无影响；
> 2).删除硬连接f2，对f1,f3也无影响；
> 3).删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效；
> 4).同时删除原文件f1,硬连接f2，整个文件会真正的被删除。



#### 文件权限怎么看（rwx）

> 具体的权限(例如777的含意等)在下面解释下：
> 1.777有3位，最高位7是设置文件所有者访问权限，第二位是设置群组访问权限，最低位是设置其他人访问权限。
>
> 其中每一位的权限用数字来表示。具体有这些权限：
>
> r(Read，读取，权限值为4)：对文件而言，具有读取文件内容的权限；对目录来说，具有浏览目录的权限。
>
> w(Write,写入，权限值为2)：对文件而言，具有新增、修改文件内容的权限；对目录来说，具有删除、移动目录内文件的权限。
>
> x(eXecute，执行，权限值为1)：对文件而言，具有执行文件的权限；对目录了来说该用户具有进入目录的权限。



#### 文件的三种时间（mtime, atime，ctime），分别在什么时候会改变

> **1>访问时间（access time 简写为atime）**文件中的数据库最后被访问的时间
>
> **2>修改时间（modify time 简写为mtime）** 文件内容被修改的最后时间
>
> **3>变化时间(change time 简写为ctime**)  文件的元数据发生变化，如权限、所有者等
>
> 2变了，3一定会变；3变了，2不一定变；



#### Linux监控网络带宽的命令，查看特定进程的占用网络资源情况命令

