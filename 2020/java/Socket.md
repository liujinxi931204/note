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

### TCP连接的建立步骤  
+ 客户端向服务端发送连接请求后，就被动的等待服务端的响应  
典型的TCP客户端要经过以下三个步骤  
1. 创建一个Socket实例：构造函数向指定的远程主机和端口建立一个TCP连接  
2. 通过套接字的I/O流域服务端通信  
3. 使用Socket类的close方法关闭连接  
+ 服务端的工作是建立一个通信终端，并被动等待客户端的连接  
典型的TCP服务端要经过以下两个步骤  
1. 创建一个ServerSocket实例并指定本地端口，用来监听客户端在该端口发送的TCP连接i请求  
2. 重复执行：  
+ 调用ServerSocket的accept()方法以获取客户端的连接，并通过其返回值创建一个Socket实例  
+ 为返回的Socket实例开启新的线程，并使用返回的Socket实例的I/O流与客户端通信；通信完成后，使用Socket类的close()方法关闭该客户端的套接字连接  
#### 实例  
```java
import javax.swing.plaf.BorderUIResource;
import javax.swing.plaf.SliderUI;
import java.awt.desktop.OpenURIEvent;
import java.io.*;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.stream.Stream;

public class SocketDemo {

    public static void main(String[] args) throws IOException {
        Socket client = new Socket("localhost", 55555);
        //从文件中获取数据
        BufferedReader bufferedReader = new BufferedReader(new FileReader("d://bw.txt"));
        //获取socket的OutputStream，用于发送数据到服务端
        OutputStream outputStream = client.getOutputStream();
        String len=null;
        while((len=bufferedReader.readLine())!=null){
            //发送数据到服务端
            outputStream.write(len.getBytes());
        }
        //关闭资源
        bufferedReader.close();
        outputStream.close();
        client.close();

    }
}
```
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;
import java.sql.ClientInfoStatus;

public class SocketServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(55555);
        Socket socket = serverSocket.accept();
        socket.setSoTimeout(10000);
        System.out.println("服务端");
        //接收客户端发送的数据
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        String len=null;
        while ((len=bufferedReader.readLine())!=null){
            System.out.println(len);
        }
        bufferedReader.close();
        socket.close();
    }
}

```
### UDP Socket  
UDP协议提供的服务不同于TCP协议的端到端服务，它是面向非连接的，属于不可靠协议，UDP套接字在使用前不需要进行连接。实际上，UDP协议只实现了两个功能：  
+ **在IP协议的基础上添加了端口**  
+ **在传输过程中可能产生的数据错误进行了检测，并抛弃了已经损坏的数据**  
Java 通过DatagramPacket类和DatagramSocket类使用UDP套接字，客户端和服务端都通过DatagramSocket的send()方法和receive()方法来发送和接收数据，用DatagramPacket来包装需要发送或者接收到的数据  
  
发送数据时，Java创建一个包含待发送信息的DatagramPacket实例，并将其作为参数传递给DatagramSocket实例的send方法；接收消息时，Java程序首先创建一个DatagramPacket实例，该实例预先分配了一些空间，并将接收到的消息存放在该空间中，然后把该实例作为参数传递给DatagramSocket实例的receive()方法  
  
在创建DatagramPacket实例时，需要注意：  
**如果该实例用来包装待接收的数据，则不指定数据来源的远程主机和端口号，只需要指定一个缓存数据的byte数组(在调用receive()方法来接收到数据后，源地址和端口号等信息会自动包含在DatagramPacket实例中)**  
**如果该实例用来包装待发送的数据，则需要指定发送到的目的地址和端口**  
  
由于UDP是无连接的，因此UDP服务端不需要等待客户端的连接请求以建立连接。另外，UDP服务端为所有通信使用同一套接字，这点与TCP服务端是不同的,TCP服务端则为每一个成功返回的accept()方法创建一个新的套接字  
  
一个典型的UDP服务端要经过下面三个步骤  
+ 创建一个DatagramPacket实例，指定本地端口号，并可以有选择地指定本地地址，此时，服务端已经准备好从任何客户端接收数据报文  
+ 使用DatagramSocket实例地recevie()方法接收一个DatagramPacket实例，当receive()方法返回时，数据报文就包含了客户端地地址，这样就知道回复信息应该发送到什么地方  
+ 使用DatagramSocket实例的send()方法向服务端返回DatagramPacket实例  
  
注意：**UDP程序在receive()方法出阻塞，直到收到一个数据报文或等待超时，** 



  


