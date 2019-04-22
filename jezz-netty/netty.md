### 操作系统分为:   
     用户空间 : 较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用
     内核空间 : 最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用
     Linux内核由系统内的所有进程共享,因此每个进程可以拥有4G字节的虚拟空间

### Linux 网络 IO 模型

#### 1.阻塞I/O模型 Blocking I/O
在进程空间中调用recvfrom,其系统调用直到数据包到达且被复制到应用进程的缓冲区中或者发生错误时才返回,在此期间会一直等待,进程从调用recvfrom开始到它返回的整段时间内都是被阻塞的

![Image text](https://github.com/Jezzlouis/jezz-middle/blob/master/jezz-images/images/netty/linux_io_1.png)

#### 2.非阻塞I/O模型 
recvfrom从应用进程到内核的时候,如果该缓冲区没有数据,就直接返回一个EWOULDBLOCK错误,一般都对非阻塞I/O模型进行轮询检查这个状态,看内核是不是有数据到来

![Image text](https://github.com/Jezzlouis/jezz-middle/blob/master/jezz-images/images/netty/linux_io_2.png)

#### 3.I/O复用模型(select，poll，epoll)
LINUX提供select/poll,进程通过一个或者多个fd操作(FD_SET, FD_CLR, FD_ISSET, FD_ZERO)传递给select或者poll调用,阻塞在select上,这样select就可以侦测多个fd操作是否处于
就绪状态,select/poll是顺序扫描fd是否就绪,而且支持的fd数量有限,因此他受到了制约,Linux还提供了一个epoll系统调用,epoll使用基于事件驱动方式代替扫描,因此性能更高,当fd就绪时,
立即回调函数rollback

![Image text](https://github.com/Jezzlouis/jezz-middle/blob/master/jezz-images/images/netty/linux_io_3.png)

#### 4.信号驱动I/O模型
首先开启套接字信号驱动I/O功能,并通过系统调用sigaction执行一个信号处理函数(此系统调用立即返回,进程继续工作,它是非阻塞的),当数据准备就绪时,就为该进程生成一个SIGIO信号,通过
信号回调通知应用程序调用recvfrom来读取数据,并通知主循环函数处理数据

![Image text](https://github.com/Jezzlouis/jezz-middle/blob/master/jezz-images/images/netty/linux_io_4.png)

#### 5.异步I/O
告知内核启动某个操作,并让内核在整个操作完成后(包括将数据内核复制到用户自己的缓冲区)通知我们,这种模型与信号驱动模型的主要区别是:信号驱动I/O由内核通知我们何时可以开始一个I/O
操作,异步I/O模型由内核通知我们I/O操作何时已经完成

![Image text](https://github.com/Jezzlouis/jezz-middle/blob/master/jezz-images/images/netty/linux_io_5.png)
    
### 零拷贝
https://mp.weixin.qq.com/s/JahuwQnDJwinV3LGCIT6pw