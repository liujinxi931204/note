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
