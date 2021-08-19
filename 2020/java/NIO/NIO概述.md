## 同步与异步  

同步和异步关注的是**消息通信机制**  

同步，就是在发出一个**调用**时，在没有得到结果之前，该**调用**就不返回。但是一旦**调用**返回，就得到返回值了。

换句话说，就是由**调用者主动等待这个调用的结果**  

异步，正好相反。**调用者在发出调用之后，这个调用就返回了，所以没有返回结果**。

换句话说，当一个异步过程调用发出之后，调用者不会立刻得到结果。而是由**被调用者**通过状态、通知来告知调用者，或者通过回调函数来处理这个调用  

## 阻塞与非阻塞  

阻塞和非阻塞关注的是**程序在等待调用结果（消息、返回值）时的状态**  

阻塞调用是指调用结果之前，当前线程会被挂起。调用线程只有在得到结果之后才返回  

非阻塞调用是指在不能立刻得到结果之前，该调用不会阻塞当前线程  

## IO分类  

### BIO  

java BIO就是传统的java IO，其相关类和接口在java.io包下

同步阻塞IO（传统阻塞型），服务器实现模式一个连接一个线程，即客户端有连接请求时服务端就会启动一个线程进行处理，如果这个连接不做任何事情，就会造成不必要的线程开销  

![BIO模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/BIO%E6%A8%A1%E5%9E%8B.png)  

### NIO  

 java NIO全称java Non-blocking IO，指的是JDK 1.4中提供了一系列改进的IO的新特性，被统称为NIO，是同步非阻塞的。NIO相关类都放在java.nio包下，并对原java.io包中的很多类进行了改写  

同步非阻塞IO（NIO）服务器实现模式为一个线程处理多个请求连接，即客户端发送的连接请求会被注册到多路复用器上，多路复用器轮询有IO请求就会进行处理  

![NIO模型](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/NIO%E6%A8%A1%E5%9E%8B.png)  

### AIO  

java AIO：异步非阻塞，AIO引入了异步通道的概念，采用了Proactor模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端启动线程取处理，一般适用于连接数较多且连接时间较长的应用  

## BIO、NIO、AIO使用场景分析 

+ BIO方式适用于**连接数较小且固定**的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4之前唯一的选择，程序较为简单容易理解  
+ NIO方式适用于**连接数目多且连接比较短**的架构，比如聊天服务器、弹幕系统、服务期间通讯等，编程比较复杂，JDK1.4开始支持  
+ AIO方式适用于**连接数目多且连接比较长**的架构，比如相册系统、充分调用OS参与并发操作，编程比较复杂，JDK7开始支持（AIO目前应用并不广泛）

## NIO概览  

![NIO三大组件](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/NIO%E4%B8%89%E5%A4%A7%E7%BB%84%E4%BB%B6.png)

### Buffer  

发送给一个通道的所有数据必须首先放到缓冲区，同样的，从通道中读取任何数据都要先读到缓冲区。也就是说，不会直接对通道进行数据读写，而是要先经过缓冲区。可以这样说，Channel提供了从文件、网络读取、写入数据的渠道，但是读取或者写入都必须经过Buffer  

![NIO中channel和buffer的关系](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/NIO%E4%B8%ADchannel%E5%92%8Cbuffer%E7%9A%84%E5%85%B3%E7%B3%BB.png)  

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读、写进程  

缓冲区主要包括以下类型  

| Buffer常用子类 |          描述          |
| :------------: | :--------------------: |
|   ByteBuffer   |  存储Byte数据到缓冲区  |
|  ShortBuffer   | 存储Short数据到缓冲区  |
|   CharBuffer   |  存储Char数据到缓冲区  |
|   IntBuffer    |  存储Int数据到缓冲区   |
|   LongBuffer   |  存储Long数据到缓冲区  |
|  DoubleBuffer  | 存储Double数据到缓冲区 |
|  FloatBuffer   | 存储Float数据到缓冲区  |

下面有一个例子来说明如何使用channel和buffer  

```java
@Test
public void test1() throws IOException {
    RandomAccessFile file = new RandomAccessFile("D://test.txt","rw");

    FileChannel channel = file.getChannel();
    //创建一个可以存储1024个Byte的缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);

    //表示将channel中的内容读取到buffer中，返回读入到缓冲区的字节数
    int data = channel.read(buffer);
    while (data!=-1){
        System.out.println("读取到的字节数："+data);
        //将buffer从读模式切换到写模式
        buffer.flip();
        while (buffer.hasRemaining()){
            System.out.print((((char) buffer.get())));
        }
        //清空buffer
        buffer.clear();
        data=channel.read(buffer);
    }
    file.close();
}
```

#### Buffer中的capacity、position和limit  

缓冲区中有三个需要熟悉的属性，分别是  

+ `capacity`：缓冲区的容量，不能为负并且不能更改  

+ `position`：缓冲区的位置，其实下一个需要读取或者写入元素的索引，不能为负并且不能大于limit  

+ `limit`：缓冲区的限制，它的值不能为负并且不能大于capacity  

另外还有一个标记`mark`

`mark`、`capacity`、`position`、`limit`需要遵守以下规则  
$$
0\leq mark\leq position \leq limit\leq capacity
$$
以下是写入和读取模式下 `position`、`limit`、`capacity`的说明  

![Buffer的三个参数](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Buffer%E7%9A%84%E4%B8%89%E4%B8%AA%E5%8F%82%E6%95%B0.png)  

`capacity`  

作为存储器块，缓冲区具有一定的固定大小，也成为"容量"。缓冲区满了以后，需要清空（读取数据或者清除），然后才能再次写入  

`position`  

将数据写入缓冲区时，可以在某个位置执行操作。`position`初始值是0，当一个数据写入缓冲区时，`position`被移动一个位置，指向缓冲区中下一个可以执行操作的位置。`position`的最大值是`capacity-1`  

`limit`  

在写入模式下，`Buffer`的`limit`表示可以写入缓冲区的数据量的限制，此时`limit=capacity`  

在读模式下，`limit`表示最多可以读取到的数据。

因此当将`Buffer`的模式从写模式切换成读模式时，`limit`被设置为之前写模式下的`position`的值，换句话说，能读取到之前所有写入的数据  

#### 分配缓冲区  

要获取`Buffer`对象，必须先分配它。每个`Buffer`类都有一个`allocate()`方法来执行此操作。  

```java
ByteBuffer  buffer = ByteBuffer.allocate(48);//创建一个容量为48个字节的缓冲区
```

#### 将数据写入缓冲区  

可以通过两种方式将数据写入缓冲区  

+ 将数据从`Channel`写入`Buffer`  

+ 通过`Buffer`的`put()`方法自己将数据写入缓冲区  

```java
int data = fileChannel.read(buffer);//将Channel的数据读入到Buffer中，返回读入到Buffer中的字节数
buffer.put(127);//将127直接写入到Buffer中
```

#### 从缓冲区读取数据  

可以通过两种方式从缓冲区读取数据  

+ 将数据从`Buffer`读取到`Channel`  

+ 通过`Buffer`的`get()`方法自己从缓冲区读取数据  

```java
int data = fileChannel.write(buffer);//将数据从Buffe写入到Channel中，返回写入到Channel中的字节数
byte data = buffer.get();//直接从Buffer中读取数据
```

#### 读写模式切换  

`filp()`方法将`Buffer`从写入模式切换到读取模式。调用`flip()`之后会将`position`设置为0，并将`limit`设置为切换之前的`position`的值。

#### rewind倒带  

`Buffer`对象的`rewind()`方法可以重新将`position`设置为0，`limit`保持不变，因此可以利用`rewind()`方法重读缓冲区的数据  

#### clear和compact  

如果调用`clear()`方法，将会设置`position`置为0，并将`limit`设置为`capacity`值。换句话说，就是`Buffer`被清空了，实际上`Buffer`中的数据没有被删除。如果在调用`clear()`方法的时候，缓冲区中有任何未读的数据，那么这些数据将会被遗忘，这意味着不再有任何标记说明读取了哪些数据，哪些数据还没有被读取  

如果缓冲区中还有数据，并且向稍后读取它，但需要先写入一些数据到缓冲区，这时候应该调用`compact()`方法，它会将所有未读的数据复制到`Buffer`的开头，然后将`position`设置为最后一个未读元素的下一个位置。`limit`属性仍然是被设置为`capacity`。

#### mark和reset  

通过调用`Buffer`对象的`mark()`方法在`Buffer`中标记指定位置，然后，可以通过`reset()`方法将`position`重新置回标记的位置  

```java
buffer.mark();
// 调用 buffer.get() 等方法读取数据...

buffer.reset();  // 设置 position 回到 mark 位置。
```

#### equals和compareTo  

可以使用`equals()`和`compareTo()`来比较两个缓冲区的大小  

`equals()`成立的条件  

+ 类型相同  

+ 在缓冲区中具有相同数量的剩余字节、字符等  

+ 所有剩余的字节、字符都相等  

`equals()`只比较缓冲区的一部分，而不是它内部的每个元素。实际上，它只比较缓冲区中的剩余元素

`compareTo()`方法比较两个缓冲区中剩余元素，在下列情况中，一个`Buffer`被视为小于另一个`Buffer`  

+ 第一个不相等的元素小于另一个`Buffer`中对应的元素  
+ 所有元素相等，但第一个`Buffer`在第二个`Buffer`之前耗尽了元素（第一个`Buffer`元素较少）

### Channel  

通道Channel是对BIO中流的模拟，可以通过它读取和写入数据  

通道与流的不同之处在于，流只能在一个方向上移动（一个流必须是InputStream或者Outputstream的子类），而通道是双向的，可以用于读、写或者同时读写  

通道主要包括以下类型  

|        类型         |                             描述                             |
| :-----------------: | :----------------------------------------------------------: |
|     FileChannel     |                      从文件中读写取数据                      |
|   DatagramChannel   |                   通过UDP读写网络中的数据                    |
|    SocketChannel    |                   通过TCP读写网络中的数据                    |
| ServerSocketChannel | 可以监听新进来的TCP连接，对每一个新机来的连接都会创建一个SocketChanne |

### 通道的聚集和分散操作  

NIO具有内置的`scatter/gather`支持，用于描述读取和写入通道的操作  

+ `scatter`：分散地从`Channel`读取是将数据读入多个`Buffer`的操作。  

+ `gather`：聚集地写入`Channel`是将来自多个缓冲区的数据写入到单个通道的操作。

通道的聚集和分散操作在需要将传输的数据分开处理的场合非常有用。例如，如果消息是由标题和正文组成，则可以将标题和正文保留在单独的缓冲区中，这样做更容易处理标题和正文  
#### `Scatter Reads`分散读取  

![NIO分散读取](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/NIO%E5%88%86%E6%95%A3%E8%AF%BB%E5%8F%96.png)  

下面演示一下如何分散读取  

```java
@Test
public void test2() throws IOException {

    //文件内容是0123456789
    RandomAccessFile file = new RandomAccessFile("D:\\test.txt", "rw");
    FileChannel channel = file.getChannel();
    ByteBuffer buffer1 = ByteBuffer.allocate(5);
    ByteBuffer buffer2 = ByteBuffer.allocate(1024);

    //一个缓冲数组，包含两个缓冲区
    ByteBuffer[] byteBuffers={buffer1,buffer2};

    //一次性将数据全部读入2个缓冲区
    long data = channel.read(byteBuffers);
    System.out.println("一共读取 "+data+" 个字节");

    System.out.println("开始读取第一个buffer");
    //读写模式的切换
    buffer1.flip();
    while (buffer1.hasRemaining()){
        System.out.print(((char) buffer1.get()));//输出01234
    }

    System.out.println();
    System.out.println("开始读取第二个buffer");
    //读写模式的切换
    buffer2.flip();
    while (buffer2.hasRemaining()){
        System.out.print(((char) buffer2.get()));//输出56789
    }
    file.close();
}
```

注意多个缓冲区首先插入到缓冲数组中，然后将数组作为参数传递个`channel.read()`方法，`read()`方法就会根据缓冲区在数组中出现的顺序从通道中写入数。一旦缓冲区满，就会将数据写入下一个缓冲区。

分散读取在移动到下一个缓冲区之前，必须先填充满前一个缓冲区，因此这种方式不适合那些不固定大小的消息  

#### `Gather Write`聚集写入  

![NIO聚集写入](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/NIO%E8%81%9A%E9%9B%86%E5%86%99%E5%85%A5.png)  

下面演示一下如何聚集写入

```java
@Test
public void test3() throws IOException {

    //从第一个文件中读取数据到buffer1
    RandomAccessFile file1 = new RandomAccessFile("D://test1.txt", "rw");
    FileChannel channel1 = file1.getChannel();
    ByteBuffer buffer1 = ByteBuffer.allocate(10);
    channel1.read(buffer1);
    buffer1.flip();

    //从第二个文件读取数据到buffer2
    RandomAccessFile file2 = new RandomAccessFile("D://test2.txt", "rw");
    FileChannel channel2 = file2.getChannel();
    ByteBuffer buffer2 = ByteBuffer.allocate(10);
    channel2.read(buffer2);
    buffer2.flip();

    //两个缓冲区的数组
    ByteBuffer[] byteBuffers = {buffer1,buffer2};
    RandomAccessFile file = new RandomAccessFile("D://test.txt", "rw");
    FileChannel channel = file.getChannel();
    //将buffer数组作为参数传递给write方法
    channel.write(byteBuffers);

    file1.close();
    file2.close();
    file.close();

}
```

注意多个缓冲区首先插入到缓冲数组中，然后将缓冲数组作为参数传递给`channel.write()方法`,`write()`方法会根据缓冲区在缓冲数组中的顺序，依次将缓冲区中数据写入到`channel`。**尤其需要注意的是，写入到`channel`中只会将`position`到`limit`之间的数据写入到`channel`中，因此聚集写可以适应大小不固定的情况**

#### `Channel to Channel`通道到通道的传输  

在NIO中，如果其中一个通道是`FileChannel`，可以直接将数据从一个通道传输到另一个通道。`FileChannel`类有一个`transferFrom()`和`transferTo()`方法  

下面是一个简单的例子  

```java
@Test
public void test() throws Exception{
    RandomAccessFile file = new RandomAccessFile("D:\\test.txt","rw");
    FileChannel fromChannel = file.getChannel();
    
    RandomAccessFile file = new RandomAccessFile("D:\\test1.txt","rw");
    FileChannel toChannel = file.getChannel();
    
    int position = 0;
    int count = fromChannel.size();
    
    fromChannel.transferTo(toChannel,position,count);
    
}
```

参数`position`和`count`分别是指写入的位置和最大的传输字节数（总数）。如果源通道的字节数小于`count`，则传输实际的字节数  

**实际上，`transferFrom`和`transferTo`这两个方法实现了NIO中的零拷贝，因此这两个方法进行拷贝会比普通的拷贝要快**  

### Selector  

选择器是一个NIO组件，它可以检测一个或多个NIO通道，并确定哪些通道可以用于读或写了。这样，单个线程可以管理多个通道，从而管理多个网络连接  

一个选择器可以对应多个通道，选择器是通过`SelectionKey`这个关键对象完成对多个通道的选择的。注册选择器的时候会返回此对象，调用选择器的`selectedKeys()`方法也会返回此对象。每一个`SelectionKey`都包含了一些必要信息，比如关联的通道和选择器，获取到`SelectionKey`后就可以从中取出对应通道操作  

#### 为什么使用选择器  

仅使用单线程来处理多个通道的优点是，只需要更少的线程来处理通道。实际上只需要一个线程来处理所有通道。对于操作系统而言，在线程之间切换是昂贵的，并且每个线程也占用操作系统中的一些资源。因此，使用的线程越少越好  

以下是使用一个`Selector`来处理3个`Channel`的线程图  

![selector](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/selector.png)  





