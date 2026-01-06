---
title: IO模型详解
tags: ['操作系统']
date: 2021/10/1 20:46:25
categories: '操作系统'
---
## I/O模型

有五种I/O模型：

1. 阻塞型I/O：blocking IO
2. 非阻塞型I/O：nonblocking IO
3. I/O复用：IO multiplexing
4. 信号驱动I/O：signal driven IO
5. 异步I/O：asynchronous IO

对于一个network IO (以read举例)，它会涉及到两个系统对象：一个是调用这个IO的进程，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：

**阶段1：**等待数据准备 (Waiting for the data to be ready)

**阶段2：** 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010165434.png)

### 阻塞型I/O

阻塞型IO就是，**数据的读入和写出都在一个线程内，需要等待其完成。**

​	当在使用阻塞IO的时候，应用程序会被无情的挂起，等待内核完成操作，因为此时的内核可能将CPU时间切换到了其它需要的进程中，在我们的应用程序看来感觉被卡主(阻塞)了。

​	采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在`while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010164243.png)

**问题：**

我们都知道重复的进行创建销毁线程是很耗费系统资源的，并且其线程的利用率也不是很高。

### 非阻塞型I/O

当使用非阻塞函数的时候，和阻塞IO类比，**内核会立即返回，返回后获得足够的CPU时间继续做其它的事情。**

当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。 从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次 发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。
**所以，用户进程第一个阶段不是阻塞的,需要不断的主动询问kernel数据好了没有；第二个阶段依然总是阻塞的。**

![NIO](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010165830.png)

### I/O复用

IO复用(IO multiplexing)，也称事件驱动IO(event-driven IO)，就是在**单个线程里同时监控多个套接字，通过 select 或 poll 轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。**

IO复用同非阻塞IO本质一样，不过利用了新的select系统调用，由内核来负责本来是请求进程该做的轮询操作。看似比非阻塞IO还多了一个系统调用开销，不过因为可以支持多路IO，才算提高了效率。

**I/O复用的特点是进行了两次系统调用，进程先是阻塞在 select/poll 上，再是阻塞在读操作的第二个阶段上。**

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010173348.png)

**select的缺点**

- select返回的是含有整个句柄的数组，应用程序需要遍历整个数组才能发现哪些句柄发生了事件
- select的触发方式是水平触发，应用程序如果没有完成对一个已经就绪的文件描述符进行IO操作，那么之后每次select调用还是会将这些文件描述符通知进程
- 内核 / 用户空间内存拷贝问题，select每次都会改变内核中的句柄数据结构集，因而每次select调用时都需要从用户空间向内核空间复制所有的句柄数据结构，产生巨大的开销
- 单个进程能够监视的文件描述符的数量存在最大限制，通常是1024，当然可以更改数量

**epoll实现**

> epoll在内核中会维护一个红黑树和一个双向链表，红黑树存放通过epoll_ctl方法向epoll对象中添加进来的事件，所以不需要每次调用epoll_wait都全量复制所有的事件结构。双向链表存放就绪的事件，所有添加到epoll中的事件都会与设备(网卡)驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个回调方法，这个回调方法在内核中叫ep_poll_callback,它会将发生的事件添加到rdlist双链表中。调用epoll_wait就会直接返回链表中的就绪事件，效率高。

**缺点：**

我们现在可以使用select/poll等系统调用进行事件的检测，但是select会返回所有建立连接的文件描述符，我们需要一个一个去遍历文件描述符，才能知道哪些数据准备好了，然后才能进行后续的操作，但是如果有10K个连接过来，然后我们调用一次select 或者poll的函数调用，然后去遍历所有的文件描述符，结果只有两个客户端的数据准备好了，那么我们剩下的9998次都是在空转，消耗了很多的系统资源。

### 信号驱动I/O

信号驱动IO与BIO和NIO最大的区别就在于，在IO执行的数据准备阶段，不会阻塞用户进程。
如下图所示：当用户进程需要等待数据的时候，会向内核发送一个信号，告诉内核我要什么数据，然后用户进程就继续做别的事情去了，而当内核中的数据准备好之后，内核立马发给用户进程一个信号，说”数据准备好了，快来查收“，用户进程收到信号之后，立马调用recvfrom，去查收数据。

![SIGIO](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010175721.png)

### 异步I/O

异步IO真正实现了IO全流程的非阻塞。用户进程发出系统调用后立即返回，内核等待数据准备完成，然后将数据拷贝到用户进程缓冲区，然后发送信号告诉用户进程**IO操作执行完毕**（与SIGIO相比，一个是发送信号告诉用户进程数据准备完毕，一个是IO执行完毕）。其流程如下：

![AIO](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010175808.png)

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/20201010175831.png)

[马士兵讲BIO、NIO、AIO](https://www.bilibili.com/video/BV11K4y1C7rm?p=1)