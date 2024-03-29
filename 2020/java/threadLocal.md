## ThreadLocal是什么  
ThreadLocal是JDK java.lang包中的一个用来实现不同线程的线程数据隔离的一个工具  
ThreadLocal这个类提供局部变量，这些变量与其他正常变量的不同之处在于，每一个访问该变量的线程在其内部都有一个独立的初始化的变量副本；ThreadLocal实例变量通常采用private static 在类中修饰  
只要ThreadLocal的变量能被访问，并且线程存活，那么每个线程都会持有ThreadLocal变量的副本。当一个线程结束的时候，它所持有的所有ThreadLocal相对应的实例副本都可被回收  
**ThreadLocal适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用(相同数据共享)，也就是变量在线程间隔离(不同的线程数据隔离)而在方法或类间共享的场景**  
    
**ThreadLocal的作用和同步机制有些相反：同步机制是为了保证多线程环境下数据的一致性；而ThreadLocal是保证了多线程环境下数据的独立性**  

## ThreadLocal类用在哪些场景  
一般来说，ThreadLocal在实际工业生产中并不常见，但是在很多框架中使用却能解决一些框架问题；比如如Spring中的事务、Spring中作用域Scope为Request的Bean使用ThreadLocal来解决  

## ThradLocal的特性  
ThreadLocal和synchornized都是为了解决多线程中相同变量的访问冲突问题，不同的点是  
+ synchornized是通过线程等待，牺牲时间来解决访问冲突  
+ ThreadLocal是通过每个线程单独一份存储空间，牺牲空间来解决冲突  
  

相比较于synchonized，ThreadLocal具有线程隔离的效果，只有在线程内才能获取到对应的值，线程外则不能访问到想要的值  

正因为ThreadLocal的线程隔离特性，所以它的应用场景更为特殊一些。当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal实现  

**但是由于ThreadLocal在每个线程中都创建了副本，所以要考虑它对资源的消耗，比如内存的占用比会比不适用ThreadLocal更大**  

## ThreadLocal的方法  
```java
//ThreadLocal是可以带泛型的

//get()方法是用来获取ThreadLocal在当前线程中保存的变量副本  
public T get();

//set()方法是用来设置当前线程中变量的副本
public void set(T value);

//remove()方法是用来移除当前线程中变量的副本
public void remove();

//initialValue()是一个protected方法，一般是用来在使用时进行重写的，如果在没有set()的时候就是用get()，会调用initialValue()方法初始化内容  
protected T initvalValue();
```
如下  
```java
void step1() {
    User u = threadLocalUser.get();
    log();
    printUser();
}

void log() {
    User u = threadLocalUser.get();
    println(u.name);
}

void step2() {
    User u = threadLocalUser.get();
    checkUser(u.id);
}
```
注意到普通的方法调用一定是同一个线程执行的，所以step1()、step2()、log()方法内，threadLocalUser.get()获取的User对象是同一个实例。**也就是说，可以利用ThreadLocal在不同方法之间传递参数**  
### 示例  
```java
public class ThreadLocalExample {
    public static class MyRunnable implements Runnable {

        private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();

        @Override
        public void run() {
            //注意这里 set的值是run函数的内部变量，如果是MyRunnable的全局变量
            //则无法起到线程隔离的作用
            threadLocal.set((int) (Math.random() * 100D));
            try {
                //sleep两秒的作用是让thread2 set操作在thread1的输出之前执行
                //如果线程之间是共用threadLocal，则thread2 set操作会覆盖掉thread1的set操作
                //从而两者的输出都是thread2 set的值
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                System.out.println(e);
            }
            System.out.println(threadLocal.get());
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyRunnable sharedRunnableInstance = new MyRunnable();

        Thread thread1 = new Thread(sharedRunnableInstance);
        Thread thread2 = new Thread(sharedRunnableInstance);

        thread1.start();
        thread2.start();

        thread1.join(); //wait for thread 1 to terminate
        thread2.join(); //wait for thread 2 to terminate
    }
}
```
输出结果  
```java
thread1 start
thread2 start
38
thread1 join
78
thread2 join
```
如果线程之间是共享ThreadLocal，则thread2的set操作会覆盖thread1的set操作，两者输出都是thread2的值，结果却看到输出不同的值，说明thread1、thread2的ThreadLocal是不共享的  
## ThreadLocal实现原理  
### set()方法  
**set()方法设置在当前线程中ThreadLocal变量的值**，该方法的源码为  
```java
public void set(T value) {
    //1. 获取当前线程实例对象
    Thread t = Thread.currentThread();
    //2. 通过当前线程实例获取到ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
        //3. 如果Map不为null,则以当前threadLocl实例为key,值为value进行存入
        map.set(this, value);
    else
        //4.map为null,则新建ThreadLocalMap并存入value
        createMap(t, value);
}
```
通过源码可以知道value是存放在了ThreadLocalMap中的，当前先把它理解为一个普普通通的Map即可，也就是说，**数据value是真正存放在了ThreadLocalMap这个容器中，并且是以当前threadLocal实例为key**  

**首先ThreadLocalMap是怎么样来的？**，源码很清楚，是通过getMap()方法进行获取的  

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```
该方法直接返回的就是当前线程对象t的一个成员变量threadLocals  
也就是说，**ThreadLocalMap的引用作为Thread的一个成员变量，被Thread进行维护**。当map为null的时候会调用createMap(t,value)方法    

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
该方法就是new一个ThreadLocalMap实例对象，然后统一以当前ThreadLocal实例作为key，值为value存放到ThreadLocalMap中，然后将该ThreadLocalMap赋值给当前线程的threadLocals  
### 总结  
**通过当前线程对象thread获取thread所维护的ThreadLocalMap，若ThreadLocalMap不为null，则以当前threadLocal实例为key，值为value的键值对存入ThreadLocalMap；若ThreadLocalMap为null，则就新建ThreadLocalMap然后再以当前threadLocal为key，值为value的键值对存入ThreadLocalMap中**  

### get()方法  
**get()方法是获取当前线程中的ThreadLocal变量的值**  

```java
public T get() {
    //1. 获取当前线程的实例对象
    Thread t = Thread.currentThread();
    //2. 获取当前线程的threadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //3. 获取map中当前threadLocal实例为key的值的entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //4. 当前entitiy不为null的话，就返回相应的值value
            T result = (T)e.value;
            return result;
        }
    }
    //5. 若map为null或者entry为null的话通过该方法初始化，并返回该方法返回的value
    return setInitialValue();
}
```
get()方法的思维与set()方法的思维正好相反。只有当ThreadLocalMap不为null且以当前ThreadLocal为key的entry不为null的时候，才可以拿到当前下线程threadLocal的变量的值  
接下来看一下setInitialValue()方法  

```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
这段代码的逻辑和set()方法几乎一致，另外值得关注一下initialValue()方法  
```java
protected T initialValue() {
    return null;
}
```
这个方法**是protected修饰的，也就是说继承ThreadLocal的子类可以重写该方法，实现赋值为其他的初始值**  
### 总结  
**get()方法通过当前线程thread实例获取到它所维护的ThreadLocalMap，然后以当前ThreadLocal实例为key获取该map中的键值对(Entry),若Entry不为null则返回Entry的value；如果获取的ThreadLocalMap为null或者Entry为null，就以当前threadLocal为key，value值为null，存入ThreadLocalMap中，然后返回null**  

### remove()方法  
```java
public void remove() {
    //1. 获取当前线程的threadLocalMap
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        //2. 从map中删除以当前threadLocal实例为key的键值对
        m.remove(this);
}
```
**删除数据当然是从map中删除数据，先获取与当前线程相关联的ThreadLocalMap然后从map中删除该threadLocal实例为key的键值对即可**  

## ThreadLocalMap详解  
从上面的分析可以知道，数据其实都是放在了ThreadLocalMap中，ThreadLocal的get、set和remove方法实际上具体是通过ThreadLocalMap的getEntry、set和remove来实现的  
### Entry数据结构  
ThreadLocalMap是ThreadLocal一个静态内部类，和大多数容器一样内部维护了一个数组，同样的，ThreadLocalMap内部维护了Entry类型的table数组  
```java
/**
 * The table, resized as necessary.
 * table.length MUST always be a power of two.
 */
private Entry[] table;
```
通过注释可以看出，table数组的长度为2的幂次方  
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
Entry是一个以ThreadLocal为key，Object为value的键值对，另外需要注意的是这里的**ThreadLocal是弱引用，因为Entry继承了WeakReference，在Entry的构造方法中，调用了super(k)方法就将threadLocal实例包装成了一个WeakReference**  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/28/1609124454075-1609124454154.png)  
### set()方法  
与concurrentHashMap、hashMap等容器一样，threadLocalMap也是采用散列表进行实现的  
#### 散列表  
理想状态下，散列表就是一个包含关键字的固定大小的数组，通过使用散列函数，将关键字映射到数组的不同位置  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/28/1609126133201-1609126133204.png)  
在理想状态下，哈希函数可以将关键字均匀的分散到数组的不同位置，不会出现两个关键字散列值相同(假设关键字数量小于数组的大小)的情况。但是在实际使用中，经常会出现多个关键字散列值相同的情况(被映射到数组的同一个位置)
，这种情况称为散列冲突。为了解决散列冲突，主要采用下面两种方式，分离链表法和开发定址法
##### 分离链表法  
分离链表法使用链表解决冲突，将散列值相同的元素都保存到一个链表中。当查询的时候，收i先找到元素所在的链表，然后遍历链表查找对应的元素，典型实现为hashMap、concurrentHashMap的拉链法  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/28/1609127022897-1609127022898.png)
##### 开放定址法  
开放定址法不会创建链表，当关键字散列到的数组单元已经被另一个关键字占用的时候，就会尝试在数组中寻找其他的单元，直到找到一个空的单元。探测数组空单元的方式有很多，最简单的线性探测法。线性探测法就是从冲突的数组单元开始，依次往后搜索空单元，如果到数组结尾，再从头部开始搜索(环形查找)  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/12/28/1609127435667-1609127435669.png)  
**ThreadLocalMap中使用开放定址法来处理散列冲突，而hashMap中使用分离链表法。之所以采用不同的方式主要是因为：在ThreadLocalMap中的散列值分散的十分均匀，很少会出现冲突。并且ThreadLocalMap经常需要清除无用的对象，使用纯数组更加方便**  

**set方法的源码**  
```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    //根据threadLocal的hashCode确定Entry应该存放的位置
    int i = key.threadLocalHashCode & (len-1);

    //采用开放地址法，hash冲突的时候使用线性探测
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //覆盖旧Entry
        if (k == key) {
            e.value = value;
            return;
        }
        //当key为null时，说明threadLocal强引用已经被释放掉，那么就无法
        //再通过这个key获取threadLocalMap中对应的entry，这里就存在内存泄漏的可能性
        if (k == null) {
            //用当前插入的值替换掉这个key为null的“脏”entry
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //新建entry并插入table中i处
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //插入后再次清除一些key为null的“脏”entry,如果大于阈值就需要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
从源码中可以清楚的看到threadLocal实例的hashCode()方法实现，该方法实际上总是用一个AtomicInteger加上0x61c88647来实现的。0x61c88647这个数是有特殊意义的，它从能够保证hash表的每个散列桶能够均匀的分布，这是Fibonacci Hashing。也正是能够均匀分布，所以threadLocal选择使用开发地址法来解决hash冲突的问题 

怎么样确定新值插入到hash表中的位置？  

该操作的源码为`key.threadLocalHashCode & (len-1)`,同hashMap和concurrentHashMap等容器的方式一样，利用当前key(即threadLocal实例)的hashCode与哈希表的大小相与，因为哈希表大小总是2的幂次方，所以相与等同于取模的过程，这样可以通过key分配到具体的哈希桶中去。而至于为什么取模要通过位与运算的原因就是位运算的执行效率远远高于取模运算  

怎么样解决hash冲突  

源码中通过`nextIndex(i, len)`方法来解决hash冲突的问题，该方法位`((i + 1 < len) ? i + 1 : 0);`,也就是不断往后探测，当到哈希表尾的时候再从0开始，呈环形  

怎么样解决"脏"Entry  

在分析threadLocal,threadLocalMap以及Entry的关系的时候，我们已经知道使用threadLocal有可能存在内存泄漏（对象创建出来后，在之后的逻辑一直没有使用该对象，但是垃圾回收器无法回收这个部分的内存），在源码中针对这种key为null的Entry称之为“stale entry”，直译为不新鲜的entry，我把它理解为“脏entry”，自然而然，Josh Bloch and Doug Lea大师考虑到了这种情况,在set方法的for循环中寻找和当前Key相同的可覆盖entry的过程中通过replaceStaleEntry方法解决脏entry的问题。如果当前table[i]为null的话，直接插入新entry后也会通过cleanSomeSlots来解决脏entry的问题

如何进行扩容  
几乎和大多数容器一样，threadLocalMap会有扩容机制，那么它的threadhold又是怎么样确定的？  
```java
 private int threshold; // Default to 0
    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

    /**
     * Set the resize threshold to maintain at worst a 2/3 load factor.
     */
    private void setThreshold(int len) {
        threshold = len * 2 / 3;
    }  
```
根据源码可知，在第一次为threadLocal进行赋值的时候会创建初始化大小为16的threadLocalMap，并且通过setThreadhold方法设置threadhold，其值为当前哈希数组长度乘以2/3，也就是说加载因子为2/3(**加载因子是衡量哈希表密集程度的参数，如果加载因子越大，说明哈希表被装载的越多，出现hash冲突的可能性越大；反之，则被装载的越少，出现hash冲突的可能性越小。同时如果过小，很显然内存使用率不高，该值取值应该考虑到内存使用率和hash冲突概率的一个平衡，如hashMap、concunrrentHashMap的加载因子都为0.75**)。这里**threadLocal初始化大小为16，加载因子为2/3**，所以哈希表可用大小为16*2/3=10，即哈希表可用容量为10  

扩容  

从set方法中可以看出当hash表的size大于threadhold的时候，会通过resize方法进行扩容  
```java
/**
 * Double the capacity of the table.
 */
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    //新数组为原数组的2倍
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            //遍历过程中如果遇到脏entry的话直接另value为null,有助于value能够被回收
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                //重新确定entry在新数组的位置，然后进行插入
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }
    //设置新哈希表的threshHold和size属性
    setThreshold(newLen);
    size = count;
    table = newTab;
}   
```
新建一个大小为数组长度的两倍的数组，然后遍历旧数组中的entry并将其插入到新的hash数组中，主要注意的是，**在扩容的过程中针对脏entry的话会令value为null，以便能够被垃圾回收器能够回收，解决隐藏的内存泄漏的问题**  
### getEntry()方法  
```java
private Entry getEntry(ThreadLocal<?> key) {
    //1. 确定在散列数组中的位置
    int i = key.threadLocalHashCode & (table.length - 1);
    //2. 根据索引i获取entry
    Entry e = table[i];
    //3. 满足条件则返回该entry
    if (e != null && e.get() == key)
        return e;
    else
        //4. 未查找到满足条件的entry，额外在做的处理
        return getEntryAfterMiss(key, i, e);
}
```
若能当前定位的entry的key和查找的key相同的话就直接返回entry，否则的话就是在set的时候存在hash冲突的情况下，需要通过getEntryAfterMiss做进一步处理  
```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            //找到和查询的key相同的entry则返回
            return e;
        if (k == null)
            //解决脏entry的问题
            expungeStaleEntry(i);
        else
            //继续向后环形查找
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```
通过nextIndex往后环形查找，如果找到和查询的key相同的entry的话就直接返回，如果在查找过程中遇到脏entry的话就是用expungeStaleEntry方法进行处理  
**为了解决潜在的内存泄露的问题，在set、resize、getEntry这些地方都会对这些脏entry进行处理，可见为了尽可能解决这个问题几乎无时无刻都在做努力**  
### remove()方法  
```java
/**
 * Remove the entry for key.
 */
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            //将entry的key置为null
            e.clear();
            //将该entry的value也置为null
            expungeStaleEntry(i);
            return;
        }
    }
}
```
该方法通过往后环形查找到与指定key相同的entry后，先通过clear方法将key置为null后，使其转换为一个脏entry，然后调用expungeStaleEntry方法将其value置为null，以便垃圾回收时能够及时清理，同时将table[i]置为null  









