### Socket简介  
socket，又称套接字，是在不同的进程之间进行网络通讯的一种协议、约定或者说规范  
   
对于socket编程，更多的时候像是基于TCP/IP等协议做的一层封装或者说抽象，是一套系统所提供的用于进程网络通信相关编程的接口  
  
协议相当于通信的程序间达成的一种约定，它规定了分组报文的结构、交换方式、包含的意义以及怎样对报文所包含的信息进行解析，TCP/IP协议族有IP协议、TCP协议和UDP协议。现在TCP/IP协议族中的主要socket类型为流套接字(使用TCP协议)和数据报套接字(使用UDP协议)  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/04/1604477750141-1604477750188.png)  
 
### TCP socket  
TCP协议提供面向连接的服务，通过它建立的是可靠的连接。Java为TCP协议提供了两个类：Socket类和ServerSocket类。一个Socket实例代表了TCP连接的一个客户端，而一个ServerSocket实例代表了TCP连接的服务端  
  
一般在TCP Socket编程中，客户端有多个，服务端只有一个  
  
客户端TCP向服务端TCP发送连接请求，服务端的ServerSocket实例则监听来自客户端的TCP连接请求，并为每个请求创建新的Socket实例，由于服务端在调用accpet()等待客户端的连接请求时会阻塞，直到收到客户端发送的来凝结请求才会继续往下执行代码，因此要为每一个Socket连接开启一个线程  
  
服务端需要同时处理ServerSocket实例和Socket实例，而客户端只需要使用Socket实例  
  
每个Socket实例会关联一个InputStream和OutputStream对象，将字节写入套接字的OutputStream来发送数据，从套接字的InputStream来接收数据  
  
#### Socket类  
Socket类：该类实现客户端套接字  
##### 构造方法  
**public Socket(String host,int port)** 
创建套接字对象并将其连接到指定主机上的指定端口号。如果指定的host为null，则相当于指定地址为回送地址  
```java
Socket client = new Socket("127.0.0.1",8888); 
```  
##### 成员方法  
+ **public InputStream getInputStream()**：返回此套接字的输入流  
     如果此Socket具有相关联的通道，则生成的InpuytStream的所有操作也关联该通道  
     关闭生成的InputStream也将关闭相关的Socket  
+ **public OutputStream getOutputStream()**：返回此套接字的输出流  
     如果此Socket具有相关联的通道，则生成的OutputStream的所有操作也关联该通道  
     关闭生成的OutputStream也将关闭相关的Socket  
+ **public void close()**：关闭此套接字  
     一旦一个Socket被关闭，它不可再使用  
     关闭此Socket也将关闭相关的InputStream和OutputStream  
+ **public void shutdownOutput()**：禁止此套接字的输出流  
     任何先前写出的数据将被发送，随后终止输出流  
+ **public void connect(SocketAddress host,int timeout) throws IOException**  
     将套接字连接到指定的主机。仅当使用无参构造函数实例化Socket时才需要此方法  
#### ServerSocket类  
ServerSocket类：该类实现了服务端套接字，该对象等待通过网络的请求  
##### 构造方法  
**public ServerSocket(int port)**
使用该构造方法在创建ServerSocket对象时，就可以将其绑定到一个指定的端口上  
```java
ServerSocket server = new ServerSocket(8888);
``` 
##### 成员方法  
+ **public Socket accept()**  
    侦听并接受连接，返回一个新的Socket对象，用于和客户端实现通信。该方法回一直阻塞直到连接建立  
+ **public int getLocalPort()**  
    返回服务器套接字正在侦听的端口  
+ **public void setTimeout(int timeout)**  
    设置服务器套接字在accept()期间等待客户端的时间的超时值  
+ **public void bind(SocketAddress host,int backlog)**  
    将套接字绑定到SocketAddress对象指定的服务器和端口  

### TCP


  


