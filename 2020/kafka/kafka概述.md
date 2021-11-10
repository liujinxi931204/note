

## Kafka简介  

Kafka是一个分布式流处理平台，一般有三个关键功能  

+ 发布和订阅记录流。在这方面，它类似于一个消息队列或者企业消息系统  

+ 持久化收到的消息流，从而具有容错能力  

+ 处理收到的记录流  

Kafka擅长哪些方面？  它主要被应用于两大类应用  

+ 构建可以在系统或者应用程序之间可靠获取数据的实时流数据管道  

+ 构建转换或者响应数据流的实时流应用程序  

首先需要明确几个概念  
+ Kafka作为一个集群运行在一个或者多个可跨多个数据中心的服务器上 

+ Kafka集群分类存储的记录流被称为主题（Topics）  

+ 每个消息记录包含一个键、一个值和时间戳  

Kafka有四个核心API  

+ 生产者API：允许应用程序发布消息至一个或多个Kafka的主题（Topics）

+ 消费者API：允许应用程序订阅一个或多个主题，并处理这些主题接收到的记录流  

+ Strams API：允许应用程序充当流处理器（stream processor），从一个或多个主题获取输入流，并生产一个输出流至一个或多个的主题，能够有效地变换输入流为输出流  

+ Connector API：允许构建和运行可重用的生产者或消费者，能够把Kafka主题连接到现有的应用程序或者数据系统。例如，一个连接到关系数据库的连接器（connector）可能会获取每个表的变化  

![image-20211110104239893](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/image-20211110104239893.png)  

Kafka的客户端和服务器之间是靠一个简单的、高性能的，与语言无关的TCP协议完成的。

## Kafka的基础架构  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/8d2ee577801a71b81bb42e2af6a8e0f9.png)  

+ Producer：消息生产者，向Kafka中发布消息的角色  

+ Consumer：消息消费者，即从Kafka中拉取消息消费的客户端  

+ Consumer Group：消费者组，消费者组则是一组中存在多个消费者，消费者消费Broker中当前Topic的不同分区的消息，消费者组之间互不影响，所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。某一个分区中的消息只能够被一个消费者组中的一个消费者所消费  

+ Broker：一台Kafka服务器就是一个Broker，一个集群由多个Broker组成，一个Broker可以容纳多个Topic  

+ Topic：主题，可以理解为一个队列，生产者和消费者都是面向一个Topic  

+ Partition：分区，为了实现扩展性，一个非常大的Topic可以分布到多个Broker上，一个Topic可以分为多个Partition，每个Partition是一个有序的队列（分区有序，不能保证全局有序）  

+ Replica：副本Replication，为保证集群中某个节点发生故障，节点上的Partition数据不丢失，Kafka可以正常的工作，Kafka提供了副本机制，一个Topic的每个分区有若干个副本，一个Leader和多个Follower  

+ Leader：每个分区多个副本的主角色，生产者发送数据的对象，以及消费者消费数据的对象都是leader  

+ Follower：每个分区多个副本的从角色，实时的从leader中同步数据，保持和leader数据的同步，leader发生故障的时候，某个follower会成为新的leader  

### Topics主题和partitions分区    

主题是一种分类或发布的一系列记录的名义上的名字。Kafka的主题始终是支持多用户订阅的，也就是说，一个主题可以有零个、一个或者多个消费者订阅写入的数据  

每个Topic可以划分为多个分区（每个Topic至少有一个分区），同一Topic下的不同分区包含的消息是不同的。每个消息在被添加到分区时，都会被分配到一个offset（称之为偏移量），它是消息在此分区中的唯一编号，Kafka通过offset保证消息在分区内的顺序，offset的顺序不跨分区，即Kafka只保证在同一个分区内的消息是有序的  

![Kafka的partition](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E7%9A%84partition.png)  

每一条消息发送到broker时，会根据partition的规则选择存储到哪一个partition。如果partition规则设置合理，那么所有的消息都会均匀的分布在不同的partition中，这样就有点类似数据库的分库分表的概念，把数据做了分片处理  

partition是以文件的形式存储在文件系统中，比如创建一个名为firstTopic的topic，其中有3个partition，那么在Kafka的数据目录（`/tmp/kafka-log`）中就有3个目录  

### 生产者和消费者组  

生产者发布数据到他们所选择的主题。生产者负责选择记录分配到主题中的哪个分区，这可以使用轮询算法进行简单地平衡复杂，也可以根据一些更复杂的语义区分（比如基于记录一些键值）来完成  

消费者以消费者组的名称来标记自己，每个发布到主题的消息都会发送给所有订阅主题的消费者组中的一个消费者实例。消费者实例可以单独的进程或单独的机器上  

如果所有的消费者实例都属于相同的消费者组，那么记录将有效的被均衡到每个消费者实例  

如果所有的消费者实例有不同的消费者组，那么每个消息将被广播到所有的消费者进程  

![Kafka消费者组](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E6%B6%88%E8%B4%B9%E8%80%85%E7%BB%84.png)  

上图是两个服务器的Kafka集群，其中具有四个分区和两个消费者组。A消费者组中有两个消费者实例，B消费者组中有四个消费者实例。这时A消费者组中每个消费者实例消费两个partition中的消息，而B消费者组中每个消费者实例消费一个partition中的消息。一般不建议消费者组中消费者实例的数量多于partition的数量，因为这样会导致消费者组中有一些消费者实例没有消息可以消费。

### 文件存储  

Kafka文件存储也是通过本地罗盘的方式存储的，主要是通过相应的log与index等文件保存具体的消息文件  

![Kafka的log](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E7%9A%84log.png)  

生产者不断的向log文件追加消息文件，为了防止log文件过大导致定位效率低下，Kafka的log默认情况下以1G为一个分界点，当`.log`文件大小超过1G的时候，此时会创建一个新的`.log`文件，同时为了快速定位大文件中的消息位置，Kafka采取了分片和索引的机制来加速定位。在Kafka存储log的地方，会存在消费的偏移量以及具体的分区信息，分区信息主要包括`.index`和`.log`文件

**如何快速定位数据？**  

`.index`文件存储的消息的offset+真实的起始偏移量，.log中存放的是真实的数据  

![Kafka日志定位](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E6%97%A5%E5%BF%97%E5%AE%9A%E4%BD%8D.png)  

如上图所示，需要定位找到offset为5378的消息，具体的过程如下  

+ 需要知道的是Kafka每一个segment的命名都是以上一个文件的最后一个offset进行命名的，所以根据二分法查找到offset为5378的消息需要先找到最大的小于5378的segment文件，也就是结尾为5376的`.index`和`.log`文件  
+ 计算5378这条消息在`.index`文件中的相对offset，即`5378-5376=2`。然后从`.index`文件中找到最大的offset小于2的这条记录，也就是`(1,0)`这条记录。即相对offset为1的这条记录在`.log`文件中的position为0  
+ 在`.log`文件中从position为0的位置开始查找，直到找到offset为5378的这条消息

