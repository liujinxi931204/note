volatile可以看作是轻量级的synchornized，它只保证共享变量的可见性。在线程A修改被volatile修饰的变量后，线程B能够读取到正确的值。java在多线程中操作共享变量的过程中，会存在指定重排序与共享变量工作内存缓存的问题  
## java内存模型  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/20/1608435333675-1608435333684.png)  
java内存模型规定了所有的变量都存储在主内存中。每条线程还有自己的工作内存，线程的工作内存中保存了被该线程所使用到的变量(这些变量是从主内存中拷贝出来的)。线程对变量的所有操作(读取、赋值)都必须在工作内存中进行。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值得传递均需要通过主内存来完成  

## 并发编程的三大概念  
### 可见性  
可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是线程修改的结果另一个线程马上就能看到。比如：用volatile修饰的变量，就会具有可见性。volatile修饰的变量**不允许线程内部缓存和重排序，即直接修改内存**，所以对其他线程是可见的。但是需要注意：**volatile只能让它修饰的内容具有可见性，但不能保证它具有原子性**。例如：  
```java
//保证变量a具有可见性
volatile int a=0;
//但是不能保证a++具有原子性
a++;
```
这个变量a具有可见性，但是a++依然是一个非原子操作，也就是说这个操作同样存在线程安全问题  
而普通的共享变量不能保证可见性，因为普通共享变量被修改以后，什么时候写入主内存是不确定的，而当其他线程去读取时，此时主内存中的变量值可能还是原来的旧值，因此无法保证可见性  
在java中synchornized和Lock也能够保证可见性，synchornized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前将对变量的修改刷新到主内存当中，因此可以保证可见性  
### 原子性  
即一个操作或者多个操作要么全部执行并且执行的过程中不会被任何因素打断，要么就都不执行。在java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行要么不执行  
比如a=0;(a非long和double类型)这个操作是不可分割的；再比如a++；这个操作实际上是a=a+1;是可分割的，所以这不是一个原子操作。非原子操作都会存在线程安全的问题，需要使用同步技术让它变成一个原子操作。java的concurrent包下提供了一些原子类，比如AtomicInteger、AtomicLong等  
java内存模型只能保证基本读取和赋值操作是原子操作，如果要实现更大范围操作的原子性，可以通过synchornized和Lock来实现  
### 有序性  
有序性就是程序执行的顺序按照代码的先后顺序执行  
什么是指令重排序，一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证最终执行结果和代码顺序的结果是一致的。指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性  
在java中，可以通过volatile关键字来保证一定的"有序性"。另外可以通过synchornized和Lock来保证有序性，很显然，synchornized和Lock可以保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码块，自然保证了有序性  

**也就是说，要想并发程序正确地地执行，必须要保证原子性、可见性和有序性。只要有一个没有被保证，就有可能会导致多线程并发执行的正确性**  
## volatile作用  
### volatile可见性  
一旦一个共享变量(类的成员变量、类的静态成员变量)被volatile修饰以后，那么就具备了两层语义  
+ 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量值，这个新值对其他线程来说是立即可见的  
+ 禁止指令重排序  
```java
//线程1boolean stop = false;while(!stop){    doSomething();}//线程2stop = true;
```
这段代码是很典型的一段代码，很多中断线程代码可能都会采用这种标记办法，但事实，线程却不一定会被中断。原因在于，当线程2更改了stop变量的值之后，但是还没有来得及下入主内存中，线程2转去做其他的事请了，那么线程1由于不知道线程2对变量stop的更新，因此会一致循环下去  
但是使用volatile修饰之后就变得不一样了  
+ 使用volatile关键字会强制将修改的值立即写入主内存中  
+ 使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存失效(反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效)  
+ 由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主内存中读取  
### 原子性  
```java
public class Test {    
    public volatile int inc = 0;     
    public void increase() {        
        inc++;    
    }     
    public static void main(String[] args) {        
        final Test test = new Test();        
        for(int i=0;i<10;i++){            
		new Thread(){                
		public void run() {                    
			for(int j=0;j<1000;j++)                      		
                                test.increase();                
                         };            
                }.start();        
        }         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完            
        Thread.yield();        
        System.out.println(test.inc);    
    }
}
```
事实上这段代码的每次运行结果都不一致，都是一个小于10000的数字。虽然volatile保证了变量inc的可见性，但是无法保证inc自增的原子性。自增操作是不具备原子性的，它包括读取变量的原始值，进行+1操作，写入工作内存，那么就是说自增操作的三个子操作可能会分割开执行
可能会出现以下情况  
假设某个时刻变量inc的值为10  
线程1对变量进行自增操作，先读取inc的原始值之后被阻塞了  
然后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，由于线程1只是对变量inc进行读取操作，没有对变量进行修改操作，所以不会导致线程2的工作内存中缓存变量无效，也不会导致主内存中值刷新，所以线程2直接去竹内存中读取inc的值，发现inc的值为10后，然后进行+1操作，并把11写入工作内存，最后写入主内存  
然后线程1接着进行加1操作，由于已经读取了inc的值，注意此时在线程1的工作内存中inc的值仍为10，所以线程1对inc进行加1操作后的inc值为11，然后将11写入工作内存，最后写入主内存  
那么两个线程分别进行了一次自增操作，inc只增加了1  

**解决方案，可以通过synchornized或Lock来加锁，也可以同AtomicInteger**  
### volatile保证有序性  
volatile关键字禁止指令重排序有两层意思  
+ 当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后的操作肯定还没有执行  
+ 在进行指令优化时，不能将在volatile变量的读操作或者写操作的语句放在其后面执行，也不把volatile变量后面的语句放到其前面执行  
```java
//x、y为非volatile变量
//flag为volatile变量 
x = 2;        //语句1
y = 0;        //语句2
flag = true;  //语句3
x = 4;         //语句4
y = -1;       //语句5
```
由于flag为volatile修饰的变量，那么在进行指令重排序的时候，不会将语句3放到语句1、语句2前面，也不会将语句3放到语句4、语句5后面。但是需要注意，语句1和语句2、语句4和语句5的顺序是不作任何保证的  
并且volatile关键字能保证，执行语句3时，语句1和语句2必定是执行完毕的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的  
```java
volatile boolean isOK = false;
//假设以下代码在线程A执行
A.init();
isOK=true;

//假设以下代码在线程B执行
while(!isOK){
  sleep();
}
B.init();
```
A线程在初始化的时候，B线程处于睡眠状态，等待A线程完成初始化的时候才能够进行自己的初始化。这里的先后关系依赖于isOk这个变量  
如果没有volatile修饰isOK这个变量，那么isOK的赋值就可能出现在A.init()之前(指令重排序)，此时A没有初始化，而B的初始化就破坏了它们之前形成的依赖关系，可能就会出错  
## volatile的实现原理  
处理器为了提高处理速度，不直接和内存进行通讯，而是将系统内部的数据读到内部缓存后进行操作，但操作完之后不知道什么时候会写入内存  

如果对声明了volatile变量进行写操作时，JVM会向处理器发送一条Lock前缀的指令，将这个变量所在缓存写入到系统内存，这一步确保了如果有其他线程对声明了volatile变量进行修改，则立即更改主内存中的数据  

但这时其他处理器的缓存还是旧的，所以在多处理器的环境下，为了保证各个处理器缓存一致，每个处理器会通过嗅探在总线上传播的数据来检查自己的缓存是否过期，当处理器发现自己缓存行对应的内存地址被修改，会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作时，会强制从系统内存把数据读到处理器缓存里，这一步确保了其他线程获得了声明了volatile变量都是从主内存中获取最新的  

Lock前缀指令实际上相当于一个内存屏障(也成为内存栅栏),它确保指令重排序时不会把其后面的指令排到内存屏障之前，也不会把前面的指令排到内存屏障之后。即在执行到内存屏障这句指令时，在它前面的操作已经全部完成  
## volatile的应用场景  
synchornized关键字是防止多个线程同时执行一段代码，那么就会很影响程序的执行效率，而volatile关键字在某些性能要求下要优于synchornized，但是要注意volatile关键字是无法替代synchornized关键字的，因为volatile关键字无法保证操作的原子性。通常来说使用volatile必须具备以下两个条件  
+ 对变量的写操作不依赖于当前值  
+ 该变量没有包含在具有其他变量的不变式中  
### 状态标志 
```java
volatile boolean shutdownRequest;

public void shutdown(){
    shutdownRequest=true;
}

public void doWork(){
    while(!shutdownRequst){
        //do stuff
    }
}
```
线程1执行doWork()的过程中，可能有另外的线程2调用了shutdown，所以boolean变量必须是volatile  
这种类型的状态标记的一个公共特点是，**通常只有一种状态转换**，shutdownRequest标志从false转换为true，然后程序停止。这种模式可以扩展到来回的状态标志，但是只有在转换周期不被察觉的情况下才能扩展(从false到true再到false)。此外，还需要某些原子状态转换机制，例如原子变量  
### 一次性安全发布  
在缺乏同步的情况下，可能会遇到某个对象引用的更新值(由另一个线程写入)和该对象状态的旧值同时存在  
这就是著名的双重检查锁定(double-checked-locking)问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象  
下面示例，其中后台线程在启动阶段从数据库加载一些数据，其他代码在能够利用这些数据时，在使用之前将检查这些数据是否曾经发布过  
```java
public class BackgroundFloobleLoader {
    public volatile Flooble theFlooble;
 
    public void initInBackground() {
        // do lots of stuff
        theFlooble = new Flooble();  // this is the only write to theFlooble
    }
}
 
public class SomeOtherClass {
    public void doWork() {
        while (true) { 
            // do some stuff...
            // use the Flooble, but only if it is ready
            if (floobleLoader.theFlooble != null) 
                doSomething(floobleLoader.theFlooble);
        }
    }
}
```
如果theFlooble引用不是volatile类型，doWrok()中的代码在解除对theFlooble的引用时，将会得到一个不完全构造的Flooble  
该模式的一个必要条件是：被发布的对象必须是线程安全的，或者是有效的不可变对象(有效的不可变意味着对象的状态在发布之后永远不会被修改)。volatile类型的引用可以确保对象的发布形式的可见性，但是如果对象的状态在发布后将发生改变，那么就需要额外的同步  
### 独立观察  
安全使用volatile的另一种简单模式是定期发布观察结果供程序内部使用。例如，假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的volatile变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值  
```java
public class UserManager {
    public volatile String lastUser;
 
    public boolean authenticate(String user, String password) {
        boolean valid = passwordIsValid(user, password);
        if (valid) {
            User u = new User();
            activeUsers.add(u);
            lastUser = user;
        }
        return valid;
    }
}
```
该模式是前面模式的扩展，将某个值发布以在程序内的其他地方使用，但是与一次性时间的发布不同，这时一系列独立事件。这个模式要求被发布的值是有效不可变的----即值的状态在发布后不会改变。使用该值的代码需要清楚该值可能随时发生变化  
### 单例模式  
```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() { 
//双重检查加锁，只有在第一次实例化时才启用同步机制，提高了性能 
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```
+ **第一次判断singleton是否为null**  
第一次判断是在synchornized同步代码块外进行判断，由于单例模式只会创建一个实例，并通过getInstance()方法返回singleton对象，所以，第一次判断，是为了在singleton对象已经创建的情况下，避免进入同步代码块，提升效率  
+ **第二次判断singletone是否为null**  
第二次判断是为了避免以下情况发生  
1. 假设：线程A已经经过第一次判断，判断singletone=null，准备进入同步代码块  
2. 此时线程B获得时间片，由于线程A并没有创建实例，所以，singleton任然等于null，所以线程B创建了实例singleton  
3. 此时，线程A再次获得时间片，由于刚刚经过第一次判断singleton=null(不会重复判断)，进入同步代码块，这个时候，如果不加第二次判断的话，那么线程A又会创造一个实例singleton，就不满租单例模式的要求，所以第二次判断非常有必要  
+ **为什么要加volatile关键字**  
既然已经使用了synchornized作为限制，为什么还要加入volatile？  
首先，需要知道volatile可以保证可见性和有序性，但是不能保证原子性  
其次，这点也很关键，就是对象的创建不是一步完成的，是一个复合操作，需要3个指令  
```java
singleton=new Singleton();
```
指令1：获取singleton对象内存地址  
指令2：初始化singleton对象  
指令3：将这块内存地址，指向引用变量singleton  
由于volatile禁止JVM对指令进行重排序，所以创建对象的过程仍然会按照指令1-2-3有序执行  
反之，如果没有volatile关键字，假设线程A正常创建一个实例，那么指令执行的顺序可能是1-3-2，当指令执行到1的时候，线程B执行getInstance()方法，这时获取到的可能就是对象的一部分或者不正确的对象  
## volatile和synchornized的区别  
+ volatile只能修饰实例和变量，而synchornized可以修饰方法和代码块  

+ volatile可以保证数据的可见性，但是不保证原子性；synchornized是一种排他互斥的机制，既保证可见性，又保证原子性  

+ volatile不会造成线程的阻塞，synchornized可能会造成线程的阻塞  

+ volatile可以看作是synchornized的轻量版，volatile不保证原子性，但是如果对一个共享变量进行多个线程的赋值，而没有其他的操作，可以使用volatile来替代synchornized，因为赋值本身是有原子性的，而volatile有保证了可见性和有序性，所以就是线程安全的  

## happens-before  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/happens-before-01.png)