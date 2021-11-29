所谓副本机制（Replication），也可以称之为备份机制，通常是指分布式系统在多台互联网的机器上保存有下相同的数据拷贝。副本机制一般有如下好处  

+ **提供数据冗余**：即使系统部分组件失效，系统依然能够继续运转，因而增加了整体可用性以及数据持久性  

+ **提供高伸缩性**：支持横向扩展，能够通过增加机器的方式来提升读性能，进而提高读操作吞吐量  

+ **改善数据局部性**：允许将数据放入与用户地理位置相近的地方，从而降低系统延时  

然而在Kafka的副本机制中，目前只能享受数据冗余带来的高可用和高持久性的好处。不过即便如此，副本机制依然是Kafka设计架构的核心所在，它也是Kafka确保系统高可用和消息持久性的重要基石  

### 副本定义  

Kafka是有主题概念的，而每一个主题又进一步划分为若干个分区。副本的概念实际上是在分区层级下定义的，每个分区配置有若干个副本。  

**所谓副本（Replica），本质就是一个只能追加写消息的提交日志**。根据Kafka副本机制的定义，同一个分区下的所有副本保存有相同的消息序列，这些副本分散保存在不同的Broker上，从而能够对抗部分Broker宕机带来的数据不可用  

实际上，每台Broker都可能保存有各个主题下不同分区的不同副本，因此，单个Broker上存有成百上千个副本的现象是非常正常的  

![Kafka副本机制](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E5%89%AF%E6%9C%AC%E6%9C%BA%E5%88%B6.png)  

上图是一个有3台Broker的Kafka集群的副本分布情况。从这张图可以看到，主题1的每个分区的副本都落在了集群中的不同Broker上，从而实现了数据的冗余  

### 副本角色  

Kafka是基于领导者的副本机制。  

在Kafka中，副本分为两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本  

Kafka的副本机制比其他的分布式系统要更严格一些。在Kafka中，追随者副本是不对外提供服务的。这就是说，任何一个追随者副本都不能响应消费者和生产者的读写请求。所有的请求都必须由领导者副本来处理，或者说，所有的读写请求都必须发往领导者副本所在的broker，由该broker负责处理。追随者副本不处理客户端请求，它唯一的任务就是从领导者副本**异步拉取**消息，并写入到自己提交的日志中，从而实现与领导者副本的同步  

当领导者副本挂掉了以后，或者说领导者副本所在的broker宕机时，Kafka依托于Zookeeper提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老leader副本重启回来以后，只能作为追随者副本加入到集群中  

![Kafka副本拉取](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/Kafka%E5%89%AF%E6%9C%AC%E6%8B%89%E5%8F%96.png)  

前面说过Kafka没能提供读操作横向扩展以及改善局部性的原因就在于，**追随者副本是不对外提供服务的**。为什么Kafka要这样设计呢？其实这种副本机制有两个好处  

1. 方便实现"Read-your-writes"  

   所谓Read-your-writes，顾名思义就是，当你使用生产者API向Kafka成功写入消息后，马上使用消费者API去读取刚才生产的消息。如果允许追随者副本堆外提供服务，由于副本同步是异步的，因此有可能出现追随者副本还没有从领导者副本那里拉取到最新的消息，从而使得客户端看不到最新写入的消息  

2. 方便实现单调读（Monotonic Reads）  

   什么是单调读？就是对于一个消费者用户而言，在多次消费消息时，它不会看到某一条消息一会存在一会不存在  

   如果允许追随者副本提供读服务，那么假设当前有 2 个追随者副本 F1 和 F2，它们异步地拉取领导者副本数据。倘若 F1 拉取了 Leader 的最新消息而 F2 还未及时拉取，那么，此时如果有一个消费者先从 F1 读取消息之后又从 F2 拉取消息，它可能会看到这样的现象：第一次消费时看到的最新消息在第二次消费时不见了，这就不是单调读一致性。但是，如果所有的读请求都是由 Leader 来处理，那么 Kafka 就很容易实现单调读一致性。

### In-sync Replicas（ISR）  

追随者副本不提供服务，只是定期异步拉取领导者副本中的数据而已。既然是异步的，就存在着不可能与leader实时同步的风险。因此我们需要明确地知道，追随者副本到底在什么条件下才算与Leader同步  

基于这个想法，Kafka引入了In-sync Replicas，也就是所谓地ISR副本集合。ISR中的副本都是与Leader同步的副本，相反，不在ISR中的追随者副本就被认为是与Leader不同步的  

首先需要明确，Leader副本天然就在ISR中。也就是说，**ISR不只是追随者副本集合，它必然包括Leader副本。甚至在有些情况下，ISR中只有Leader这一个副本**  

事实上，判断一个追随者副本是不是与Leader副本同步的标准就是Broker端的参数**replica.lag.time.max.ms**，这个参数的含义是，Follower副本能够落后Leader副本的最长时间间隔，当前默认值是10秒。这就是说，只要一个Follower副本落后Leader副本的时间不连续超过10秒，那么Kafka就认为该Follower副本与Leader副本是同步的，即使此时Follower副本中保存的消息明显少于Leader副本中的消息  

Follower 副本唯一的工作就是不断地从 Leader 副本拉取消息，然后写入到自己的提交日志中。如果这个同步过程的速度持续慢于 Leader 副本的消息写入速度，那么在 replica.lag.time.max.ms 时间后，此 Follower 副本就会被认为是与 Leader 副本不同步的，因此不能再放入 ISR 中。此时，Kafka 会自动收缩 ISR 集合，将该副本“踢出”ISR。

值得注意的是，倘若该副本后面慢慢地追上了 Leader 的进度，那么它是能够重新被加回 ISR 的。这也表明，ISR 是一个动态调整的集合，而非静态不变的

### 生产者确认机制  

生产者确认机制又称为Kafka的ack机制，指的是producer的消息发送确认机制，这直接影响到Kafka集群的吞吐量和消息可靠性。而吞吐量和可靠性就像硬币的两面，两者不可兼得，只能平衡

Kafka为用户提供了三种可靠级别，用户根据可靠性和延迟的要求进行权衡选择不同的配置  

+ 0，producer不等待broker的ack，这一操作提供了最低的延迟，broker接收到还没有写入磁盘就已经返回，**当broker故障时有可能丢失数据**  

+ 1，producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将丢失数据。（**只是leader落盘**）

+ -1，producer等待broker的ack，partition的leader和ISR的follower全部落盘成功才返回ack，但是如果在follower同步完成后，broker发送ack之前，如果leader发生故障，会造成数据重复。(这里的数据重复是因为没有收到，所以继续重发导致的数据重复)  

### Kafka的LEO和HW（高水位）  

在副本同步中，LEO和HW用来记录数据同步处理过程的状态  

在Kafka中，高水位的作用主要有2个  

1. 定义消息可见性，即用来标识分区下的哪些消息是可以被消费者消费的  

2. 帮助kafka完成副本同步  

![kafka高水位](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/kafka%E9%AB%98%E6%B0%B4%E4%BD%8D.png)  

在分区高水位以下的消息被认为是已提交消息，反之就是未提交消息。消费者只能消费已提交消息  

高水位和LEO是副本对象的两个重要属性。Kafka所有副本都有对应的高水位和LEO值，而不仅仅是Leader副本。只不过Leader副本比较特殊，Kafka使用Leader副本的高水位来定义所在分区的高水位。换句话说，**分区的高水位就是其Leader副本的高水位**  

### 高水位更新机制  

实际上，除了保存一组高水位值和LEO值之外，在Leader副本所在的broker上，还保存了其他Follower（也称远程副本）的LEO值  

![kafka远程副本](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/kafka%E8%BF%9C%E7%A8%8B%E5%89%AF%E6%9C%AC.png)  

在这张图中，可以看到，Broker 0上保存了某分区的Leader副本和所有副本的LEO值，而Broker 1上保存了该分区的某个Follower副本的LEO值。而这些远程的副本的HW和LEO可以帮助Leader副本确定其高水位  

更新机制如下表  

| **更新对象**                  | **更新时机**                                                 |
| ----------------------------- | ------------------------------------------------------------ |
| Broker 1 上 Follower 副本 LEO | Follower 副本从 Leader 副本拉取消息，写入到本地磁盘后，会更新其 LEO 值 |
| Broker 0 上Leader 副本 LEO    | Leader副本接收到生产者发送的消息，写入到本地磁盘后，会更新其LEO值 |
| Broker 0 上远程副本 LEO       | Follower副本从Leader副本拉取消息时，会告诉Leader副本自己从哪个位移处开始拉取。Leader副本会使用这个位移值来更新远程副本的LEO |
| Broker 1 上Follower副本高水位 | Follower副本成功更新完LEO之后，会比较其LEO值与Leader副本发来的高水位值，并用两者的较小值去更新它自己的高水位 |
| Broker 0上Leader副本高水位    | 主要有两个更新时机: 一个是Leader副本更新其LEO之后;另一个是更新完远程副本LEO之后。具体的算法是:取 Leader副本和所有与Leader同步的远程副本LEO中的最小值 |

**Leader副本**  

处理生产者请求的逻辑  

+ 写入消息到本地磁盘  

+ 更新分区高水位  

  + 获取Leader副本所在Broker端保存的所有远程LEO值（LEO-1,LEO-2,LEO-3...,LEO-n）

  + 获取Leader副本高水位值：currentHW  

  + 更新currentHW=min(currnetHW,min(LEO-1,LEO-2,LEO-3...,LEO-n))

处理Follower副本拉取消息的逻辑  

+ 读取磁盘（或页缓存）中的消息数据  

+ 使用Follower副本发送请求中的位移值更新新远程副本LEO值  

+ 更新分区高水位值（具体步骤与处理生产者请求的步骤相同）  

**Follower副本**  

从Leader拉取消息的处理逻辑  

+ 写入消息到本地磁盘  

+ 更新LEO值

+ 更新高水位值  

  + 获取Leader发送的高水位值：currentHW 

  + 获取步骤2中更新过的LEO值：currentLEO  

  + 更新高水位为min(currentHW,currnetLEO)  

    
