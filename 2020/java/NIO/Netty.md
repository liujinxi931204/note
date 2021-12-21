## Netty概述  

### 原生NIO存在的问题  

1. `NIO`的类库和`API`繁杂，使用麻烦：需要熟练掌握`Selector`、`ServerSocketChannel`、`SocketChannel`、`ByteBuffer`等  

2. 需要具备其他的额外技能：需要熟悉`Java`多线程编程，因为`NIO`涉及到`Reactor`模式，必须对多线程和网络编程非常熟练，才能编写出高质量的`NIO`程序  

3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等  

4. `JDK NIO`的`Bug`：例如臭名昭著的`Epoll Bug`，它会导致`Selector`空轮询，最终导致`CPU 100%`。直到`JDK1.7`版本该问题仍旧存在，没有被根本解决  

### Netty简介   

官网地址为：https://netty/io

`Netty is an asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.  `

Netty是一个提供异步事件驱动（asynchronous event-driven）的网络应用框架，是一个用以快速开发高性能、可扩展协议的服务器和客户端。换句话说，Netty是一个`NIO`客户端服务器框架，使用它可以快速简单地开发网络应用程序，比如服务器和客户端的协议。Netty 大大简化了网络程序的开发过程比如 TCP 和 UDP 的 socket 服务的开发

“快速和简单”并不意味着应用程序会有难维护和性能低的问题，Netty 是一个精心设计的框架，它从许多协议的实现中吸收了很多的经验比如 FTP、SMTP、HTTP、许多二进制和基于文本的传统协议.因此，Netty 已经成功地找到一个方式,在不失灵活性的前提下来实现开发的简易性，高性能，稳定性。

有一些用户可能已经发现其他的一些网络框架也声称自己有同样的优势，所以你可能会问是 Netty 和它们的不同之处。答案就是 Netty 的哲学设计理念。Netty 从开始就为用户提供了用户体验最好的 API 以及实现设计

![Netty架构图](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Netty%E6%9E%B6%E6%9E%84%E5%9B%BE.png)  

### Netty的优点  

Netty对`JDK`自带的`NIO`的`API`进行了封装，解决了上述原生`NIO`存在的问题  

1. 设计优雅：适用于各种传输类型的统一`API`阻塞和非阻塞`Socket`；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型——单线程、一个或多个线程池  

2. 使用方便：详细记录的`Javadoc`，用户指南和示例；没有其他依赖项，`JDK5 (Netty 3.x)`或`JDK6 (Netty 4.x)`就足够了  

3. 高性能、吞吐量高：延迟更低；减少资源消耗；最小化不必要的内存复制  

4. 安全：完整的`SSL/TLS`和`StartTLS`支持  

5. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现`Bug`可以被及时修复，同时，更多的新功能会被加入  

### Netty版本说明  

1. Netty的版本分为：`Netty 3.x`、`Netty 4.x`和`Netty 5.x`  
2. 因为`Netty 5.x`出现重大`Bug`，已经被官网废弃了，目前推荐使用的是`Netty 4.x`的稳定版本  
3. 目前在官网可下载的版本 `Netty 3.x`、`Netty 4.0.x` 和 `Netty 4.1.x`

### Netty高性能架构设计  

#### 线程模型基本介绍  

1. 不同的线程模型，对程序的性能有很大影响。目前存在的线程模型有：传统阻塞`I/O`服务模型和`Reactor`模式  

2. 根据`Reactor`的数量和处理资源池线程的数量不同，有3中典型的实现  

   1. 单`Reactor`单线程  

   2. 单`Reactor`多线程  

   3. 主从`Reactor`多线程  
   
3. `Netty`线程模式（`Netty`主要基于主从`Reactor`多线程模型做了一定的改进，其中主从`Reactor`多线程模型有多个`Reactor`）  

#### 传统阻塞I/O服务模型  

![传统阻塞IO服务模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%BC%A0%E7%BB%9F%E9%98%BB%E5%A1%9EIO%E6%9C%8D%E5%8A%A1%E6%A8%A1%E5%9E%8B.png)  

传统阻塞I/O服务模型，对于每一个连接在服务端都需要一个独立的线程完成数据的输入、业务处理、数据返回。这样会存在两个问题  

1. 当并发数量很大，就会创建大量的线程，占用很多的系统资源  

2. 连接创建以后，如果当前线程暂时没有数据可读，该线程会阻塞在`Handler`对象中的`read`操作，导致上面的处理线程资源浪费  

#### Reactor模式  

![Reactor模式](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Reactor%E6%A8%A1%E5%BC%8F.png)  

对上图的说明：  

1. `Reactor`模式，通过一个或多个输入同时传递给服务处理器`(ServiceHandler)`的模式（基于事件驱动）  

2. 服务器端程序处理传入的多个请求，并将他们同步分派到相应的处理线程，因此`Reactor`模式也叫做`Dispatcher`模式  

3. `Reactor`模式使用`IO`复用监听事件，收到事件后，分发给某个线程（进程），这点就是网络服务器高并发处理的关键  

4. 原先有多个`Handler`阻塞，现在只有一个`ServiceHandler`阻塞  

`Reactor（也就是那个ServiceHandler）`在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理线程来对`IO`事件做出反应。就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人  

`Handlers(处理线程EventHandler)`处理线程执行`IO`事件要完成的实际事件，类似于客户想要交谈的公司的实际官员。`Reactor`通过调度适当的处理线程来响应`IO`事件，处理程序执行非阻塞操作  

根据`Reactor`的数量和处理资源池线程的数量不同，有3种典型的实现  

1. 单`Reactor`单线程  

2. 单`Reactor`多线程  

3. 主从`Reactor`多线程  

#### 单Reactor单线程模型  

![单Reactor单线程模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%8D%95Reactor%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.png)  

`Select`是之前`IO`复用模型中的标准网络编程`API`，可以实现应用程序通过一个阻塞对象监听多路连接请求  

`Reactor`对象通过`Select`监控客户端请求事件，收到请求事件后通过`Dispatch`进行分发  

如果是建立连接的请求，则由`Acceptor`通过`Accept`处理连接请求，然后创建一个`Handler`对象处理链接完成后的后续业务  

如果不是连接请求，则`Reactor`会分发调用连接对应的`Handler`来响应  

`Handler`会完成`read`→业务处理→`send`的完整业务流程  

缺点  

1. 只有一个线程，无法完全发挥多核CPU的性能。
2. 单线程很容易成为性能瓶颈。`Reactor`处理耗时长的业务逻辑时，整个进程无法处理别的连接事件或者请求事件

#### 单Reactor多线程模型  

![单Reactor多线程模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%8D%95Reactor%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.png)  

`Reactor`对象通过 `Select` 监控客户端请求事件，收到事件后，通过 `Dispatch` 进行分发  

如果是建立连接请求，则由 `Acceptor` 通过 `Accept` 处理连接请求，然后创建一个 `Handler` 对象处理完成连接后的各种事件  

如果不是连接请求，则由 `Reactor` 分发调用连接对应的 `handler` 来处理（也就是说连接已经建立，后续客户端再来请求，那基本就是数据请求了，直接调用之前为这个连接创建好的`handler`来处理）  

`handler` 只负责响应事件，不做具体的业务处理（这样不会使`handler`阻塞太久），通过 `read` 读取数据后，会分发给后面的 `worker` 线程池的某个线程处理业务。【业务处理是最费时的，所以将业务处理交给线程池去执行】  

`worker` 线程池会分配独立线程完成真正的业务，并将结果返回给 `handler`  

`handler` 收到响应后，通过 `send` 将结果返回给 `client`  

缺点：  

1. `Reactor`承担所有的事件的监听和响应，因为它是单线程运行，所以在高并发场景中容易成为性能瓶颈。也就是说，`Reactor`线程承担了过多的事请  

#### 主从Reactor多线程模型  

![主从Reactor多线程模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E4%B8%BB%E4%BB%8EReactor%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.png)  

`Reactor`主线程 `MainReactor` 对象通过 `select` 监听连接事件，收到事件后，通过 `Acceptor` 处理连接事件  

当 `Acceptor`处理连接事件后，`MainReactor` 将连接分配给 `SubReactor`  

`subreactor` 将连接加入到连接队列进行监听，并创建 `handler` 进行各种事件处理  

当有新事件发生时，`subreactor` 就会调用对应的 `handler` 处理  

`handler` 通过 `read` 读取数据，分发给后面的 `worker` 线程处理  

`worker` 线程池分配独立的 `worker` 线程进行业务处理，并返回结果  

`handler` 收到响应的结果后，再通过 `send` 将结果返回给 `client`  

`Reactor` 主线程可以对应多个 `Reactor` 子线程，即 `MainRecator` 可以关联多个 `SubReactor`  

缺点：  

1. 编程复杂度极高  

#### 小结  

1. 单`Reactor`单线程，就像前台接待员和服务员是同一个人，全程为顾客服务  

2. 单`Reactor`多线程，就像有一个前台接待员和多个服务员，接待员只负责接待  

3. 主从多`Reactor`多线程，就像有多个前台接待员和多个服务员  

#### 优点  

1. 响应快，不必为单个同步时间所阻塞，虽然`Reactor`本身依然是同步的，但是有多个`SubReactor`，即使第一个`SubReactor`被阻塞了，还可以调用下一个`SubReactor`  

2. 可以最大程度的避免复杂的线程以及同步问题，并且避免了多线程/进程的切换开销  

3. 扩展性好，可以方便的通过增加`Reactor`实例个数来充分利用CPU资源  

4. 复用性好，`Reactor`模型本身与具体事件处理逻辑无关，具有很高的复用性  

   
