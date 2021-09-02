@[toc]

本文分为三个部分：
1. 由Reactor模型理解muudo的网络库模型
2. base部分的部分理解
3. net部分的部分理解


# Reactor
Reactor 模式，市面上常见的开源软件很多都采用了这个方案，比如 Redis、Nginx、Netty 
![image](https://user-images.githubusercontent.com/40709975/131463703-0fd7e90a-433d-4f7e-aede-609b0775bc69.png)
## 演进
1. 一个线程处理一个连接，处理完毕之后释放线程
2. 资源复用，创建线程池，一个线程处理多个连接，线程会阻塞在read操作上，解决方式：
    1. 将socket改为非阻塞，不断轮询，但是消耗CPU资源
    2. I/O多路复用，当连接上有数据时，才发起请求。


对 I/O 多路复用作了一层封装，让使用者不用考虑底层网络 API 的细节，只需要关注应用代码的编写。这种模式就是Reactor（对事件反应）模式。该模式又叫
Dispatcher模式。


Reactor 模式主要由 Reactor 和处理资源池这两个核心部分组成，它俩负责的事情如下：
* Reactor 负责监听和分发事件，事件类型包含连接事件、读写事件；
* 处理资源池负责处理事件，如 read -> 业务逻辑 -> send；

Reactor和处理资源池都可以有单个或者多个，有三种模式比较经典：

> 单 Reactor 单进程 / 线程；
> 单 Reactor 多线程 / 进程；
> 多 Reactor 多进程 / 线程；

## Reactor
### 单 Reactor 单进程 / 线程
![image](https://user-images.githubusercontent.com/40709975/131463776-09e5a84b-09e4-44d6-89eb-7a62326879c2.png)
Reactor对象：监听和分发事件
Acceptor对象：获取连接
Handler对象：处理业务

缺点

>  第一个缺点，因为只有一个进程，无法充分利用多核 CPU 的性能；
>  第二个缺点，Handler 对象在业务处理时，整个进程是无法处理其他连接的事件的，如果业务处理耗时比较长，那么就造成响应的延迟；

### 单 Reactor 多线程 / 多进程
单Reactor多线程：
![image](https://user-images.githubusercontent.com/40709975/131463884-16aa83db-0113-4c17-8108-bf18223201f3.png)

Handler通过read读取到数据后，会将数据发送个子线程里的Processor对象进行业务处理。子线程里的Processor对象进行业务处理，处理完毕之后将结果发送给主线程的Handler对象，由Handler调用send方法将响应结果发送给client

优势：
> 能够充分利用多核CPU

缺陷：
> 共享资源的竞争.
> 单Reactor模式还有一个缺点，就是一个Reactor对象承担所有事件的监听和响应，在面对瞬间高并发的场景时，容易成为性能瓶颈.

单Reactor多进程：
实现起来比单Reactor多线程麻烦，父子之间的双向通行。多线程的共享数据比多进程间的通信的复杂度低。实际中也见不到单Reactor多进程的模式。

### 多 Reactor 多进程 / 线程
![image](https://user-images.githubusercontent.com/40709975/131463929-ece426ed-71a3-4408-8e5f-bcb8bcd7be10.png)

方案详细介绍：
> * 主线程的MainReactor对象通过I/O多路复用监控建立连接事件，收到事件后通过Acceptor建立连接。将连接分配给子线程。
> * 子线程的SubReactor收到主线程发来的事件后，将该事件加入到子线程的I/O多路复用中继续监控，并创建一个Handler来进行响应。
> * 当已连接的套接字有事件发生时，会调用对应的Handler来处理：read——>业务处理——>send

实现比单Reactor多进程/线程简单：
* 主线程和子线程分工明确，主线程只负责接收新连接，子线程负责后续的业务处理。
* 主线程和子线程之间的交互简单，只需要主线程把新来的连接传给子线程。
## Proactor
Reactor是非阻塞的同步网络模式，而Proactor是异步网络模式
先来看看阻塞和非阻塞：
### 阻塞
当用户调用read函数，线程会被阻塞，一直到数据准备好，并把数据从内核缓冲区拷贝到应用程序的缓冲区，当拷贝完成，read才会返回。
![image](https://user-images.githubusercontent.com/40709975/131463980-43551c6d-f66d-4403-bb07-1d2247221dbc.png)
### 非阻塞
非阻塞的read请求在数据未准备好的时候立即返回，可以继续往下执行，应用程序应该不断地轮询内核，直到数据准备好，将数据从内核缓冲区拷贝到应用程序缓冲区，read调用才可以获取到结果。
![image](https://user-images.githubusercontent.com/40709975/131464018-a7ccffa2-d18f-41b4-8e76-e6afbf65640b.png)
注意最后一次read调用，获取数据的过程是一个同步的过程，是需要等待的过程。这里的同步是指的是内核态的数据拷贝到用户程序缓冲区的这个过程。

再来看异步I/O：
### 异步I/O
【内核数据准备好】和【数据从内核态拷贝到用户态】都不需要等待。
![image](https://user-images.githubusercontent.com/40709975/131464063-80ddaafa-4f69-4182-a2fb-ca7a9a3e1fd7.png)

发起 aio_read （异步 I/O） 之后，就立即返回，内核自动将数据从内核空间拷贝到用户空间，这个拷贝过程同样是异步的，内核自动完成的，和前面的同步操作不一样，应用程序并不需要主动发起拷贝动作。

Proactor 正是采用了异步 I/O 技术，所以被称为异步网络模型。

## Reactor和Proactor的区别
Reactor是非阻塞同步网络模式，感知的是就绪可读写事件
Proactor是异步网络模式，感知的是已完成的读写事件

## muduo中的Reactor
muduo的Reactor，采用的是多Reactor多线程的模型，但是有一些差异

Reactor时序图：
![image](https://user-images.githubusercontent.com/40709975/131464123-1b84dc81-64d3-4b42-b7d4-ba93fff5e019.png)


 muduo 的整体风格受到 Netty 的影响，整个架构依照 Reactor 模式，基本与如下图所示相符：
![image](https://user-images.githubusercontent.com/40709975/131464145-a1168bd9-6b80-4a02-a70c-3e2d55931f80.png)

## muduo的线程模型：

muduo 默认是单线程模型的，即只有一个线程，里面对应一个 EventLoop。这样整体对于线程安全的考虑可能就比较简单了，
但是 muduo 也可以支持主从reactor模式：
### 主从Reactor模式

主从 reactor 是 Netty 的默认模型，一个 reactor 对应一个 EventLoop。主 Reactor 只有一个，只负责监听新的连接，accept 后将这个连接分配到子 Reactor 上。子 Reactor 可以有多个。这样可以分摊一个 Eventloop 的压力，性能方面可能会更好。如下图所示：
![image](https://user-images.githubusercontent.com/40709975/131464169-db2373b2-1d90-410b-ac54-4c6a02f133c2.png)
其中主 Reactor 的 EventLoop 就是 TcpServer 的构造函数中的 EventLoop* 参数。Acceptor 会在此 EventLoop 中运行。

而子 Reactor 可以通过 TcpServer::setThreadNum(int) 来设置其个数。因为一个 Eventloop 只能在一个线程中运行，所以线程的个数就是子 Reactor 的个数。

如果设置了子 Reactor，新的连接会通过 Round Robin 的方式分配给其中一个 EventLoop 来管理。如果没有设置子 Reactor，则是默认的单线程模型，新的连接会再由主 Reactor 进行管理。

但其实这里似乎有些不合适的地方：多个 TcpServer 之间可以共享同一个主 EventLoop，但是子 Eventloop 线程池却不能共享，这个是每个 TcpServer 独有的。不过 Netty 的主 EventLoop 和子 Eventloop 池都是可以共享的。
# 常见并发模型（各自优缺点）
## iterative服务器
## process-per-connection
## thread-per-connetion

IO多路复用，复用的是线程


# base
如何实现一个不能被继承的类：

final、使用友元、私有构造函数、虚继承等方式可以使一个类不能被继承
## Singleton
实现懒汉单例，用pthread_once()函数保证初始化函数只会在本进程中执行一次。如果在单例中有函数no_destroy()，程序结束时就不会通过atexit()函数注册清理函数，会造成内存泄漏

## ThreadLocal
线程本地变量（线程特定变量，线程私有变量）
pthread_key_create
pthread_key_delete
pthread_key_getspecific
pthread_key_setspecific

## ThreadLocalSingleton
线程本地懒汉单例，用的__thread。
每个线程都有自己的一份单例，利用pthread_key_create管理单例的生命周期，生成单例之后利用pthread_key_create传入的destructor析构该单例对象。

# net

muduo的I/O模型采用非阻塞模式，避免阻塞在read()或write()或其他系统调用上
## Socket
KeepAlive：https://zhuanlan.zhihu.com/p/28894266
socket阻塞和非阻塞有哪些不同
> 1. 建立连接
>>阻塞方式下，connect首先发送SYN请求到服务器，当客户端收到服务器返回的SYN的确认时，则connect返回，否则的话一直阻塞。
>
>>非阻塞方式，connect将启用TCP协议的三次握手，但是connect函数并不等待连接建立好才返回，而是立即返回，返回的错误码为EINPROGRESS,表示正在进行某种过程。

> 2. 接收连接
>> 阻塞模式下调用accept()函数，而且没有新连接时，进程会进入睡眠状态，直到有可用的连接，才返回。
> 
>> 非阻塞模式下调用accept()函数立即返回，有连接返回客户端套接字描述符，没有新连接时，将返回EWOULDBLOCK错误码，表示本来应该阻塞。

muduo使用的是非阻塞IO，和IO多路复用结合起来的一般都是非阻塞。
封装了socket套接字编程，诸Listen/bind/accept方法等等，有的是调用SocketOps.h的接口

## SocketOps
对Socket.h方法的补充和填充，实际调用系统接口，实现了create/bind/listen/accept/connect/read/write/close/shutdown等函数

需要关注非阻塞connect和close-on-exec的写法：
```
void setNonBlockAndCloseOnExec(int sockfd)
{
  // non-block
  int flags = ::fcntl(sockfd, F_GETFL, 0);
  flags |= O_NONBLOCK;
  int ret = ::fcntl(sockfd, F_SETFL, flags);
  // FIXME check

  // close-on-exec
  flags = ::fcntl(sockfd, F_GETFD, 0);
  flags |= FD_CLOEXEC;
  ret = ::fcntl(sockfd, F_SETFD, flags);
  // FIXME check

  (void)ret;
}
```

## InetAddress
对sockaddr_in和sockaddr_in6的封装，方便构造，获取ip地址和port

## Poller

负责监听事件是否触发的部分，在 muduo 中叫做 Poller

基类实现了hasChannel函数，判断Map是否拥有此Channel
```
 protected:
    typedef std::map<int, Channel*> ChannelMap;	
    ChannelMap channels_;	
 
 private:
     EventLoop* ownerLoop_;
```

Poller使用一个map来存放描述符fd和对应的Channel类型的指针，
这样我们就可以通过fd很方便的得到Channel了，该map是protected变量。

私有成员是一个EventLoop的指针，用来指向当前EventLoop，用来判断防止Poller被跨线程调用。

该类要被PollPoller和EPollPoller继承

## PollPoller
封装了高级IO：poll
PollPoller的数据成员有：
```
private:
     typedef std::vector<struct pollfd> PollFdList;
     PollFdList pollfds_;//存放需要监听的事件，pollfd结构体中有三个字段：fd、events、revents
```
这是一个存放pollfd的数组，用来传入poll模式中的第一个事件集合参数：
 ```
 int numEvents = ::poll(&*pollfds_.begin(), pollfds_.size(), timeoutMs);
```
事件的fd的修改技巧：
不直接修改为-1，而是改为-fd-1。方便后续还原（还原是为了去map中寻找对应的channel），还原方式仍为：-fd-1


## EPollPoller
封装了高级IO：epoll
muduo的epoll采用的是水平触发

EpollPoller和PollPoller的主要区别是Epoll的index和operation一一对应，而Poll的index是数组下标
```
private:
    static const int kInitEventListSize = 16;

    typedef std::vector<struct epoll_event> EventList;

    int epollfd_;
    EventList events_;//存放返回的就绪事件
```
sokect()：创建listenfd
bind()：将listenfd和服务器IP地址和端口号绑定
listen()：让listenfd处于监听状态
accept()：当有客户端发起请求时，服务socket变得可读


socket()
bind()
connect()




## Channel
不管是poll还是epoll，最终的事件都会存放到channel中
index在poll中表示下标，在epoll中表示三个标志位（新增、删除、更新）



## 定时器


为什么网络编程中需要定时器呢？

在开发Linux网络程序时，通常需要维护多个定时器，如维护客户端心跳时间、检查多个数据包的超时重传等。如果采用Linux的SIGALARM信号实现，则会带来较大的系统开销，且不便于管理。

timerfd是Linux为用户程序提供的一个定时器接口。这个接口基于文件描述符，通过文件描述符的可读事件进行超时通知，所以能够被用于select/poll的应用场景，采用文件描述符实现定时有利于统一事件源。
 
mudo的定时器由三个类实现，TimerId，Timer，TimerQueue，用户只能看到第一个类，其它两个类都是内部实现细节。

### TimerId
TimerId被设计用来取消Timer的，它的结构很简单，只有一个Timer指针和其序列号。其中还声明了TimerQueue为其友元，可以操作其私有数据。

### Timer
Timer是对定时器的高层次抽象，封装了定时器的一些参数，例如超时回调函数、超时时间、超时时间间隔、定时器是否重复、定时器的序列号。其函数大都是设置这些参数，run()用来调用回调函数，restart()用来重启定时器（如果设置为重复）。

### TimerQueue

TimerQueue 采用了最简单的实现（链表）来管理定时器，它的效率比不上常见的 binary heap 的做法，如果程序中大量（10 个以上）使用重复触发的定时器，或许值得考虑改用更高级的实现。

TimerQueue的接口很简单，只有两个函数addTimer()和cancel()。它的内部有channel，和timerfd相关联。添加新的Timer后，在超时后，timerfd可读，会处理channel事件，之后调用Timer的回调函数；在timerfd的事件处理后，还有检查一遍超时定时器，如果其属性为重复还有再次添加到定时器集合中。
![image](https://user-images.githubusercontent.com/40709975/131464258-15ccbb33-d61e-4b1d-8867-d460cea7fa3b.png)

TimeQueue的优化：
用二叉搜索树(例如std::set/std::map)，把Timer按到期时间先后排好序，其操作的复杂度是O(logN)，从而快速地根据当前时间找到已经到期的Timer，也要能高效地添加和删除Timer。
但我们使用时还要处理两个Timer到期时间相同的情况(map不支持key相同的情况)，做法如下：
> 两种类型的set，一种按时间戳排序，一种按Timer的地址排序
> 实际上，这两个set保存的是相同的定时器列表


定时器主要是在EventLoop中使用，EventLoop中为我们提供了四个函数，供用户使用

## Buffer
Non-blocking IO 的核心思想是避免阻塞在 read() 或 write() 或其他 IO 系统调用上，这样可以最大限度地复用 thread-of-control，让一个线程能服务于多个 socket 连接。IO 线程只能阻塞在 IO-multiplexing 函数上，如 select()/poll()/epoll_wait()。这样一来，应用层的缓冲是必须的，每个 TCP socket 都要有 stateful 的 input buffer 和 output buffer。


 TcpConnection必须要有output buffer ，如果发送的数据比较多，TCP窗口已经装不下了，此时如果不阻塞的话，就需要把剩余的未发送的数据存储起来，此时就可以交给muduo网络库来接管，网络库把它保存在output buffer里，然后注册POLLOUT事件，一旦socket变得可写就立刻发送数据。
程序只要调用 TcpConnection::send() 就行了，网络库会负责到底
 
 
 
TcpConnection必须要有input buffer TCP是一个无边界的字节流协议，接收方必须要处理“收到的数据尚不构成一条完整的消息”和“一次收到两条消息的数据”等等情况。

网络库在处理“socket可读”事件的时候，必须一次性把socket中数据读完（从操作系统buffer搬到应用层buffer），否则会反复触发POLLIN事件，造成busy loop。

所有 muduo 中的 IO 都是带缓冲的 IO (buffered IO)，你不会自己去 read() 或 write() 某个 socket，只会操作 TcpConnection 的 input buffer 和 output buffer。更确切的说，是在 onMessage() 回调里读取 input buffer；调用 TcpConnection::send() 来间接操作 output buffer，一般不会直接操作 output buffer。

* 对外表现为一块连续的内存(char*, len)，以方便客户代码的编写。
* 其 size() 可以自动增长，以适应不同大小的消息。它不是一个 fixed size array (即 char buf[8192])。
* 内部以 vector of char 来保存数据，并提供相应的访问函数


Buffer 其实像是一个 queue，从末尾写入数据，从头部读出数据。

muduo::net::Buffer 不是线程安全的，这么做是有意的，原因如下：

muduo的缓冲区Buffer类不是线程安全的，因为它是每个连接私有的，不需要锁的操作
1. 对于 input buffer，onMessage() 回调始终发生在该 TcpConnection 所属的那个 IO 线程，应用程序应该在 onMessage() 完成对 input buffer 的操作，并且不要把 input buffer 暴露给其他线程。这样所有对 input buffer 的操作都在同一个线程，Buffer class 不必是线程安全的。
2. 对于 output buffer，应用程序不会直接操作它，而是调用 TcpConnection::send() 来发送数据，后者是线程安全的。

## Acceptor
>Acceptor用于accept(2)接受TCP连接。
>Acceptor的数据成员包括Socket、Channel。
>Acceptor的socket是listening socket（即server socket）。
>Channel用于观察此socket的readable事件，并Acceptor::handleRead()，后者调用accept(2)来接受连接，并回调用户callback。

不过，Acceptor类在上层应用程序中我们不直接使用，而是把它封装作为TcpServer的成员。


在监听套接字可读事件触发时，我们会调用accept接受连接。如果此时注册过回调函数，就执行它。如果没有就直接关闭！

>成员变量 idleFd_(::open("/dev/null", O_RDONLY | O_CLOEXEC))                                           //预先准备一个空闲文件描述符
>如果已用文件描述符过多，accept会返回-1，我们构造函数中注册的idleFd_就派上用场了。当前文件描述符过多，无法接收新的连接。但是由于我们采用LT模式，如果无法接收，可读事件会一直触发。那么在这个地方的处理机制就是，关掉之前创建的空心idleFd_，然后去accept让这个事件不会一直触发，然后再关掉该文件描述符，重新将它设置为空文件描述符。
>
>这种机制可以让网络库在处理连接过多，文件描述符不够用时，不至于因为LT模式一直触发而产生坏的影响。
```
    if (errno == EMFILE)                                      //太多的文件描述符
    {
      ::close(idleFd_);
      idleFd_ = ::accept(acceptSocket_.fd(), NULL, NULL);
      ::close(idleFd_);
      idleFd_ = ::open("/dev/null", O_RDONLY | O_CLOEXEC);
    }
```

## EventLoop

EventLoop是初始分发器，其实就是一个reactor角色

负责事件循环的部分在 muduo 被命名为 EventLoop

该类中有一个loop函数，执行了该函数的线程就是I/O线程，负责调用poll来监控文件描述符，当有事件发生时，会去调用对应的回调函数，有四种回调函数：读、写、出错和关闭。这些函数在Channel中。


EventLoop是初始分发器，其实就是一个reactor角色.

I/O线程比较灵活。当I/O线程不忙的时候，也就是poll监控的文件描述符的事件不怎么发生，I/O线程还能去执行计算任务：doPendingFunctors();

queueInLoop和runInloop函数的区别：
* 前者仅是加入等待队列pendingFunctors_，等待执行
* 后者，如果本线程是I/O线程，就直接执行，如果不是I/O线程就放入等待队列pendingFunctors_


> 白色三角，继承
黑色菱形，聚合

![image](https://user-images.githubusercontent.com/40709975/131464311-d738a373-8181-46a6-ae71-c76b456df296.png)

## EventLoopThread
关于EventLoopThread有以下几点：

1. 任何一个线程，只要创建并运行了EventLoop，都称之为I/O线程。

2. I/O线程不一定是主线程。I/O线程中可能有I/O线程池和计算线程池。

3. muduo并发模型one loop per thread + threadpool

4. 为了方便使用，就直接定义了一个I/O线程的类，就是EventLoopThread类，该类实际上就是对I/O线程的封装。

（1）EventLoopThread创建了一个线程。
（2）在该类线程函数中创建了一个EventLoop对象并调用EventLoop::loop，调用EventLoopThread::startLoop()，就启动了线程也调用了loop

## EventLoopThreadPool 
EventLoopThreadPool 是一个线程池，只不过该线程池有一点特殊，该线程池中的每一个线程都要执行EventLoop进行文件描述符的监听。

此时一个线程用于管理分配线程池中的EventLoop,如果线程池为空，主线程的EventLoop用与监听所有的文件描述符baseLoop_

>每个muduo网络库有一个事件驱动循环线程池EventLoopThreadPool
每个线程池中有多个事件驱动线程EventLoopThread
每个线程运行一个EventLoop事件循环
每个EventLoop事件循环包含一个io复用Poller，一个计时器队列TimerQueue
每个Poller监听多个Channel，TimerQueue其实也是一个Channel
每个Channel对应一个fd，在Channel被激活后调用回调函数
每个回调函数是在EventLoop所在线程执行
所有激活的Channel回调结束后EventLoop继续让Poller监听


## TcpConnection

该类起到一个承上启下的作用，维持着TcpServer, Channel, Socket等等之间的联系。

TcpConnection里面的五个回调函数都会在TcpClient和TcpServer里面设置。


该类对象，客户端和服务器都会用到 。这是一个接口类。

TcpConnection在构造函数中开启了保活机制。

## TcpServer
就是一个服务器。
该类即支持单线程，也支持多线程。
![image](https://user-images.githubusercontent.com/40709975/131514420-3cacd96a-780d-4119-9396-bdad8471e205.png)

TcpServer的构造函数最少要传三个参数：EventLoop，InetAddress，string。还有第四个参数，这个参数给了默认值，默认为0，还可以给1，默认值是在构造Acceptor对象的时候，不开启ReusePort，为1的时候则去开启。
![image](https://user-images.githubusercontent.com/40709975/131514741-1079b0ec-cd39-49ff-ba03-c420b5626473.png)

注意这个loop，可以看到该loop还参与了Acceptor和EventLoopThreadPool的初始化：
![image](https://user-images.githubusercontent.com/40709975/131515032-2cd2808a-b6bd-418d-939a-731634424bdc.png)

Acceptor的初始化中，又用该loop初始化了Channel：
![image](https://user-images.githubusercontent.com/40709975/131515172-a96fad28-2d4b-410e-8bd8-1b678d69d21f.png)

EventLoopThreadPool的初始化需要传一个loop来标明baseloop是谁：
![image](https://user-images.githubusercontent.com/40709975/131516099-0f25941d-17b8-4add-b315-b950bf764c16.png)
线程池初始化成功后可以调用一个线程池初始化回调函数，这个函数要由用户手动设置，muduo的测试用例没有设置。

由此可见baseloop（主Reactor）的TcpServer，Acceptor，EventLoopThreadPool和Channel的loop都是同一个。
start：
Acceptor::listen----->acceptChannel_.enableReading()------>loop_->updateChannel(this)------>poller_->updateChannel(channel)

### 建立连接：
![image](https://user-images.githubusercontent.com/40709975/131464408-950f8d35-9506-4b8c-af79-799614f05722.png)
![TcpServer](https://user-images.githubusercontent.com/40709975/131464486-f64a6ad0-b5e4-4b69-ac8b-4845955dcf5b.png)

![image](https://user-images.githubusercontent.com/40709975/131644376-a287a90a-3dbb-42bc-8e5c-01e3f756a56b.png)

### 关闭连接：
muduo断开连接的方式：
1. 被动关闭：即对方先关闭连接，本地read()返回0，触发关闭逻辑，调用TcpConnection::handleClose()
2. 主动关闭：本地调用forceClose()，调用TcpConnection::handleClose()
3. Channel监听到POLLHUP事件，并且没有POLLIN事件，调用channel的closeCallback_


* Channel的close回调就是TcpConnection中的handleClose（在TcpConnection构造函数中设置）
* HandleRead调用readFd的返回值是0，会调用handleClose
* forceCloseInLoop会调用handleClose

handleClose-->clsoeCallback_（在TcpServer的newConnection中设置）-->removeConnection-->removeConnectionInLoop-->connectDestroyed

* TcpServer的析构函数，会直接调用connectDestroyed函数

![image](https://user-images.githubusercontent.com/40709975/131464551-ff7668ab-63da-42be-af7c-92ac8defd5ee.png)

TcpConnection的removeConnection，erase将这个连接对象从列表中移除。按照正常的思路，我们还应该将这个对象销毁掉，但是在这里我们不能立即销毁这个连接对象，如果销毁了这个对象，TcpConnection所包含的Channel对象也就跟着销毁了。而当前正在调用这个Channel对象的handleEvent函数，而这个Channel对象又销毁了，就会出现coredump。因而这个不能销毁TcpConnection对象，也就是说TcpConnection对象的生存期应该长于HandleEvent函数，如何做到这一点，可以利用shared_ptr来管理TcpConnection对象。

当连接到来，创建一个TcpConnection对象，立刻用shared_ptr来管理，这时候引用计数为1。

在Channel中维护一个weak_ptr(tie_)，将这个shared_ptr对象赋值给tie_，因为是弱引用，所以引用计数不会加1。

当连接关闭，调用了Channel的handleEvent函数，在这个函数中，将tie_提升，得到一个shared_ptr对象，此时引用计数为2。



## TcpClient

TCPClient使用Conneccor发起连接, 连接建立成功后, 用socket创建TcpConnection来管理连接。 每个TcpClient class只管理一个TcpConnecction，而TcpServer管理多个TcpConnection。

不管是TcpServer还是TcpClient，都只关注可读事件，如果Socket出错，会变为可读，如果要写入数据，先直接调用write，如果没有写完，再去关注可写事件。

后续的事件，数据等都在TcpConnection这个结构中维护。

建立连接：
![TcpClient](https://user-images.githubusercontent.com/40709975/131464614-1988605e-88e7-449a-a5b1-6d30b7b8a366.png)

关闭连接和TcpServer类似，都和TcpConnection有关，有区别的是Connector

## HttpContext
解析buffer，生成HttpRequest


http请求储存在buffer里面，每次处理一行都要往后挪动读指针
```
const char* crlf = buf->findCRLF()
buf->retrieveUntil(crlf + 2)//+2是把\r\n字符算进去

std::equal(start, end, const char*)比较函数
```

## HttpRequest

1. 第一行：请求行（首行），以空格为单位，请求方法    请求URL  HTTP协议的版本
2. 第二行到空行之前：请求报头（Header），以行为单位陈列，key：value
3. 空行
4. 请求正文(Body)

![image](https://user-images.githubusercontent.com/40709975/131464680-df3fcb92-5917-4f93-b289-01d03d0674dc.png)

## HttpResponse
对于HTTP响应的封装，封装的状态码只有5个：0,200,301,400,404

1. 第一行：状态行（首行）【版本号】+【状态码】+【状态码解释】 
2. 第二行到空行之前：响应报头（Header），遇到空行表示响应报头结束
3. 空行
4. 响应正文（网页）Body：空行之后，允许为空字符串，如果有数据，那么在响应报头会有一个Content-Length属性来标识正文的长度。如果服务器返回一个html页面，那么html页面内容就是在正文中。html告诉你是什么，根据css文件来决定图片咋放，动态效果是gs产生的

![image](https://user-images.githubusercontent.com/40709975/131464722-fcbfa80d-81c9-4743-bd52-28f8ab7b8f39.png)

## HttpServer
封装了一下TcpServer，实现的比较简单
![image](https://user-images.githubusercontent.com/40709975/131464747-7d462a90-1591-47ed-a62b-4011dd1cfb8f.png)

HTTP请求头：http://tools.jb51.net/table/http_header
HTTP管道化，队头阻塞，管道化/非管道化：https://blog.csdn.net/fesfsefgs/article/details/108294050

# muduo优化：
1. 就绪事件是按时间放在就绪队列里的，并没有做优先级区分
2. muduo多线程模型优化：为每个连接的每个请求也新建线程去处理，而不是在同一个线程中

