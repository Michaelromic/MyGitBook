# Linux:IO多路复用select,poll,epoll

https://www.cnblogs.com/Anker/p/3265058.html

《如果这篇文章说不清epoll的本质，那就过来掐死我吧》
https://zhuanlan.zhihu.com/p/63179839
https://zhuanlan.zhihu.com/p/64138532
https://zhuanlan.zhihu.com/p/64746509



> select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。
> select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的。
> 异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

* #### select
    + 基本原理：
        + select函数监视的文件描述符分为3类，分别为write_fds、read_fds、except_fds。
        + 调用select函数后，进程会阻塞，直到描述符就绪（有数据 可读、可写、或者有except），或者超时(timeout指定等待时间，如果立即返回设为null即可)，函数返回。
        + 当select函数返回后，可以通过遍历fd_set，来找到就绪的描述符。

    + 调用过程：
        1. 调用select函数，进程阻塞，然后使用copy_from_user从用户空间拷贝fd_set到内核空间 (fd是文件描述符，或者是socket句柄)
        2. 注册回调函数__pollwait
        3. 遍历所有fd，调用其对应的poll方法（对于socket，这个poll方法是sock_poll，sock_poll根据情况会调用到tcp_poll,udp_poll或者datagram_poll）
        4. 以tcp_poll为例，其核心实现就是__pollwait，也就是上面注册的回调函数。
        5. __pollwait的主要工作就是把current（当前进程）挂到设备的等待队列中，不同的设备有不同的等待队列，对于tcp_poll来说，其等待队列是sk->sk_sleep（注意把进程挂到等待队列中并不代表进程已经睡眠了）。在设备收到一条消息（网络设备）或填写完文件数据（磁盘设备）后，会唤醒设备等待队列上睡眠的进程，这时current便被唤醒了。
        6. poll方法返回时会返回一个描述读写操作是否就绪的mask掩码，根据这个mask掩码给fd_set赋值。
        7. （接3）如果遍历完所有的fd，还没有返回一个可读写的mask掩码，则会调用schedule_timeout使调用select的进程（也就是current）进入睡眠。当设备驱动发生自身资源可读写后，会唤醒其等待队列上睡眠的进程。如果超过一定的超时时间（schedule_timeout指定），还是没人唤醒，则调用select的进程会重新被唤醒获得CPU，进而重新遍历fd，判断有没有就绪的fd。
        8. 把fd_set从内核空间拷贝到用户空间。

    + 总结select的几大缺点：
        1. 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大 (上面步骤1)
        2. 同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大 (上面步骤3-6)
        3. select支持的文件描述符数量太小了，默认是1024


* ### poll
poll的实现和select非常相似，只是描述fd集合的方式不同，poll使用pollfd结构而不是select的fd_set结构，突破了select中描述符数目的限制，其他的都差不多。

* ### epoll
select和poll都只提供了一个函数——select或者poll函数。而epoll提供了三个函数，epoll_create,epoll_ctl和epoll_wait。
epoll_create是创建一个epoll句柄；epoll_ctl是注册要监听的事件类型；epoll_wait则是等待事件的产生。

> 1. 对于第一个缺点，epoll的解决方案在epoll_ctl函数中。每次注册新的事件到epoll句柄中时（在epoll_ctl中指定EPOLL_CTL_ADD），会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝。epoll保证了每个fd在整个过程中只会拷贝一次。

> 2. 对于第二个缺点，epoll的解决方案不像select或poll一样每次都把current轮流加入fd对应的设备等待队列中，而只在epoll_ctl时把current挂一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调函数，而这个回调函数会把就绪的fd加入一个就绪链表）。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd（利用schedule_timeout()实现睡一会，判断一会的效果，和select实现中的第7步是类似的）。

> 3. 对于第三个缺点，epoll没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。

---

# 我的理解：
select的调用过程：
1、假设现在有一个进程A，A的代码中定义了3个fd，分别是 fd1、fd2、fd3。
2、然后A进程会调用select()函数，操作系统就会把A从工作队列中移除，加入到3个fd各自的等待队列中。表现为A被阻塞，不会往下执行代码，也不会占用CPU资源。
3、任何一个fd就绪或者超时(就绪：有数据 可读、可写、或者有except，超时：超过等待时间），中断程序将唤起进程A。
4、当进程A被唤醒后，它知道至少有一个fd接收了数据(或者超时)。程序只需遍历一遍fd列表，就可以得到就绪的fd。

select的缺点：
1、每次调用select都要讲进程加入到所有fd的等待队列，每次唤醒又都要从各个等待队列移除。而且每次都要将整个fds列表传递给内核，有一定的开销。
2、正是因为遍历操作开销大，出于效率的考量，才会规定select的最大监视数量，默认只能监视1024个fd。
3、进程被唤醒后，程序并不知道哪些fd收到数据，还需要遍历一次。

epoll的调用过程：
1、当某个进程调用epoll_create方法时，内核会创建一个eventpoll对象。eventpoll对象含有一个 就绪队列rdlist。
2、还是假设有3个fd，通过epoll_ctl方法添加fd1、fd2、fd3的监视时，内核会将eventpoll添加到这三个fd的等待队列中。
3、当fd收到数据后，中断程序会给eventpoll的 就绪队列 添加fd引用。
eventpoll对象相当于是fd和进程之间的中介，fd的数据接收并不直接影响进程，而是通过改变eventpoll的就绪列表来改变进程状态。
4、当程序执行到epoll_wait时，如果rdlist已经引用了fd，那么epoll_wait直接返回，如果rdlist为空，阻塞进程。
5、eventpoll同样有一个等待队列，进程A的程序执行到epoll_wait时，如果rdlist为空，会将进程A放入eventpoll的等待队列中，阻塞进程。
6、当fd接收到数据时，中断程序一方面修改rdlist，另一方面唤醒eventpoll等待队列中的进程，进程A再次进入运行状态。
也是因为rdlist的存在，进程A可以知道哪些fd发生了变化。
7、程序可能随时调用epoll_ctl添加监视fd，也可能随时删除。当删除时，若该fd已经存放在就绪列表中，它也应该被移除。
所以就绪列表应是一种能够快速插入和删除的数据结构。双向链表就是这样一种数据结构，epoll使用双向链表来实现就绪队列（对应上图的rdllist）。


epoll的改进点：
1、select调用之后每次收到数据，都会返回，下次再调用select()又要把所有的fd从用户态发到内核态。
而epoll只用 调用epoll_create创建eventpoll对象 和 调用epoll_ctl添加fd监听，之后只需要在合适的时机调用epoll_wait即可，这样可以获得rdlist中所有就绪的fd了。
一方面减少了fd多次传递给内核的开销，因为会由rdlist来保存就绪的fd，而不用返回给进程。
另一方面减少了进程遍历fd列表的开销，因此进程可以直接从rdlist中得知哪些fd是就绪状态。(select是进程知道某个fd就绪了，但不知道具体是哪一个，只能遍历一次)
2、epoll没有最大支持文件描述符数量的限制。它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。