  ![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/HashMap.png)   

### hashmap的数据结构  

在JDK1.8中，HashMap是由数组+链表+红黑树构成  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/java3-1594777381.png)  

当一个值要存储到HashMap中的时候需要根据Key的值计算出它的hash，通过hash值来确认存放到数组中的位置，如果发生hash冲突就以链表的形式存储，当链表过长，HashMap会把这个链表转化成红黑树来存储  

### 源码中有一些基本的属性  

```java
/**
     *1向左移位4个，00000001变成00010000，
     * 也就是2的4次方为16，使用移位是因为移位是计算机基础运算，
     * 效率比加减乘除快。
     */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
     * 最大容量为2的30次方
     */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
     *加载因子大小，为扩容所使用
     */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
     * 当链表转为红黑树的阀值
     */
static final int TREEIFY_THRESHOLD = 8;

/**
     *红黑树转为链表的阀值
     */
static final int UNTREEIFY_THRESHOLD = 6;

/**
     *当整个hashMap元素超过64时，才有可能转为红黑树
     */
static final int MIN_TREEIFY_CAPACITY = 64;

/**
     * 存储元素的数组
     */
transient Node<K,V>[] table;

/**
     * 将数据转换成set的另一种存储形式，这个变量主要用于迭代功能
     */
transient Set<Map.Entry<K,V>> entrySet;

/**
     * 元素数量
     */
transient int size;


transient int modCount;

/**
     * 临界值，也就是元素达到这个数量时会进行扩容
     *
     * @serial
     */

int threshold;

/**
     * 加载因子，这个是个变量
     *
     * @serial
     */
final float loadFactor;  
```

总结  

+ 默认初始化容量是16，默认负载因子是0.75  

+ threshold=数组长度*loadFactor，当元素个数超过threshold（容量阈值）时，HashMap会进行扩容操作  

### 构造函数  

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //如果给的自定义容量大于最大容量就设置为最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    //设置阈值
    //tableSizeFor的作用就是生成比传入参数要大的并且是2的次方的数
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {
    //指定初始容量
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    //指定加载因子为默认值
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}


public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

在构造函数中阈值调用了tableSizeFor方法，这个方法就是找到大于等于传进来的容量的最小的2的幂次。例如，传进来的参数是9，那么这个方法的返回值就是16；如果传进来的参数是16，那么这个方法的返回值就是16。其实这个方法也就决定了HashMap中的table的长度总是2的幂次  

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

当数字本身就是2的幂次时，如果不进行减1操作，那么经过运算不会得到它本身，而是会得到比它大的最接近它的2的幂次，没有把自身包含进去，这样时不行的，仅仅靠一个减1操作，就可以规避这个bug  

### put方法  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/java2-1610525389.png)    

这是put的大致流程  

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

putVal方法的主要核心参数就是通过扰动函数计算出来的hash值，以及key、value，这个扰动函数的作用就是增加hash的散列性，随机性，减少hash碰撞  

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

在这个方法中h=key.hashCode()，无符号右移16位，就是将h的高16位右移到低16位，高位补0，让hashCode的高16位也参与运算，增加随机性  

这里选择异或运算，应该考虑到与运算结果更多的偏向0，而或运算结果更多的偏向0，从而导致结果中的0、1分布不是特别均匀  

![img](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/java5-1610525389.png)  

接着来看putVal方法  

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    //tab：引用当前hashMap的散列表
    // p:当前散列表的元素
    // n:数组的长度
    // i:表示数组的下标位置
    Node<K,V>[] tab; Node<K,V> p; int n, i;

    if ((tab = table) == null || (n = tab.length) == 0)
        //延迟初始化，懒加载
        n = (tab = resize()).length;

    //第一种情况：如果当前下标位置的桶位刚好是空的，那就直接创建node将k,v存入其中即可

    if ((p = tab[i = (n - 1) & hash]) == null)
        //用于计算在table中的位置，等价于hash%n
        tab[i] = newNode(hash, key, value, null);
    else {
        // e：n不为null的时候，找到了一个与当前要插入元素key一致的元素
        // k:表示临时的key
        Node<K,V> e; K k;
        //表示当前桶位的头元素与要插入的元素key相同，后续会进行值替换的操作

        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;

        //红黑树的情况
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //链表的情况
            for (int binCount = 0; ; ++binCount) {
                //迭代到链表最后一个元素，也没有找到要插入元素的key相同的情况
                //加入到链表末尾
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //达到树化条件，进行树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //树化操作
                        treeifyBin(tab, hash);
                    break;
                }
                //说明找到相同key的元素需要进行替换操作
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //put方法会返回该key的旧值，如果不存在旧值，就返回null

        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

这里i = (n - 1) & hash就是为了求得该hash值在table数组中的位置，因为n始终都是2的幂次，所以(n-1)&hash等价于hash%n，不过位运算的速度比较快，所以这里使用了位运算的方式。这也是HashMap中table的长度始终是2的幂次的一个优点  

结合着那个流程图，就很好理解了  

第一步判断table数组是否为空，为空就通过resize()方法进行初始化，可以看到HashMap的初始化是懒加载的思想，并不是HashMap初始化的时候就对table数组进行初始化，而是在第一次调用put方法的时候  

tab[i=(n-1)&hash]，根据hash值计算数组下标位置，如果当前下标位置没有元素，那就直接插入，有的话比较当前桶位头元素的key是否与要插入的key相等，相等就替换  

在不满足上述条件的情况下，就要判断当前桶位是链表还是红黑树，是红黑树就用红黑树的方式插入  

如果是链表，那就进入for循环遍历整个链表，比较是否有相等的key。如果有就会替换，如果没有就会遍历到最后一个节点，然后将要插入的值插入到链表的结尾。插入完成后还要看是否达到树化的条件，即链表的长度是否达到了8，达到就调用treeifyBin()方法，进行树化操作  

最后判断元素是否达到要扩容的阈值，达到就进行扩容操作  

### treeifyBin方法  

这里只截取了部分的代码，想说明的是即使链表的长度超过了8也不一定会进行树化。这里还需要判断table数组的长度是否超过了64，如果超过了64才会进行树化；不超过会进行扩容操作

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
    //还需要判断table数组的长度是否超过了64
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
```

### resize方法  

```java
final Node<K,V>[] resize() {
    //引用扩容前的哈希表
    Node<K,V>[] oldTab = table;
    //扩容前的table长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //  扩容前的阈值
    int oldThr = threshold;
    //newCap:扩容之后的数组大小
    //扩容之后的阈值
    int newCap, newThr = 0;
    // 如果 扩容前数组长度大于0
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }

        //新的数组长度等于扩容前的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            //新的阈值的等于扩容前的两倍
            newThr = oldThr << 1; // double threshold
    }
    //进入这里说明构造hashmap的函数是HashMap(int initialCapacity)或者HashMap(int initialCapacity, float loadFactor)
    //扩容前的阈值大于0
    //oldThr是大于等于initialCapacity最近的2的幂次
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
        //进入这里说明使用的是无参的构造函数，使用默认的参数
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //扩容之前，table不为null
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //说明当前桶中有数据，具体是单个数据，链表，红黑树，还未知
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                //第一种情况，当前桶位只有一个元素，从未发生过碰撞，直接计算出当前元素放在新数组的位置
                // 然后放进去即可
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;

                //当前节点已经树化
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);

                // 当前节点是链表
                else { // preserve order

                    //低位链表：存放的是在扩容之后数组下标位置与扩容前数组下标位置一致
                    Node<K,V> loHead = null, loTail = null;
                    //高位链表:存放的是扩容之后数组下标位置为当前数组下标位置+扩容之前数组长度
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;

                        //hash 0 1111
                        //hash 1 1111
                        //oldCap 10000
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    //如果低位链表不等于空
                    if (loTail != null) {
                        loTail.next = null;
                        //将链表头结点放入新数组桶中，但下标位置还是原来的位置
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        //高位链表链表头放入到新数组，下标位置为扩容前数组下标位置加扩容前的数组长度
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

扩容的方法总体来说做了三件事  

+ 计算新桶数组的容量Cap以及新的阈值threshold，扩容就是cap以及threshold都扩容到原来的两倍  

+ 根据新的数组容量创建新的数组  

+ 将老数组的数据转移到新数组  

这里重点说一下转移数组的操作  

如果当前桶中只有一个元素，那就直接计算其在新数组的下标，然后插入即可  

  newTab[e.hash & (newCap - 1)]=e;  

如果当前桶中不止有一个元素，那就需要判断这个桶位的数据是链表还是红黑树。如果是红黑树，就调用红黑树的split方法进行转移  

如果是链表，链表中的节点在新数组中的下标只有两种可能，要么在原来的位置，要么在原来数组下标+原来数组大小的位置

例如hash值为1和17的两个节点，在原来数组长度16的情况下，其在原来数组的下标都是1。但是扩容以后，数组的长度变为32，那么他们在新数组的下标就变为1和17  

在源码中可以看到，创建了两个Node，一个是低位链表，一个是高位链表  

低位链表是e.hash & oldCap==0,这些节点重新计算位置后，还是会在原来的位置  

高位链表是e.hash & oldCap==1,这些节点重新计算位置后，会在原来的位置+原来数组大小的位置  

### get方法  

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    //当前hashMap散列表
    Node<K,V>[] tab;
    //first 桶位中的头元素
    //e ：临时元素
    Node<K,V> first, e;
    //数组长度
    int n;
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //第一种情况：定位出来的桶位元素即为咱们要get的数据
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;

        // 说明当前桶位不止一个元素，可能是链表，也可能是红黑树
        if ((e = first.next) != null) {

            // 第二种情况： 当前桶位是红黑树
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            //第三种情况：桶位是链表
            do {
                //进行do while循环，一一遍历找到要查找的元素
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

get方法主要就是看getNode方法  

+ 定位出来的桶元素的头节点就是要的数据，那就直接获取然后返回  

+ 如果桶位不止一个元素，那么就有可能是红黑树也有可能是链表  

+ 如果是红黑树就按照红黑树的方法进行获取  

+ 如果是链表，就遍历整个链表，找到要的元素  

### remove方法  

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}


final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    // 引用当前hashMap的散列表
    Node<K,V>[] tab;
    //当前Node的头节点
    Node<K,V> p;
    // n:表示散列表的数组长度
    // index:表示数组下标位置
    int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        // node: 要查找的元素
        // e:当前node的下一个元素
        Node<K,V> node = null, e; K k; V v;
        // 第一种情况：当前桶位的元素即为你要删除的元素
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null &&  key.equals(k))))
            node = p;



        else if ((e = p.next) != null) {

            // 第二种情况 p是红黑树
            if (p instanceof TreeNode)
                // 红黑树查找操作
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {

                // 第三种情况 是链表的情况

                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 判断node不为空，说明按照key查找到需要删除的数据了
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            // 第一种情况 结点为树结点，需要进行树结点移除操作
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);

            // 第二种：桶位头元素即为查找结果，则将该元素的下一个元素放到桶位中
            else if (node == p)
                tab[index] = node.next;
            else
                // 因为Node为查找结果，且为p的下一个元素，
                // 因此只需要将p的下一个元素指向查找结果node的下一个元素即可
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

remove方法可以分为两个步骤，第一个步骤是找到要删除的数据，第二个步骤是进行删除  

通过hash值找到所在桶的位置，然后进行查找，查找也分为三个步骤，先看是不是当前桶位的头节点元素，如果不是就需要看这个桶位是链表还是红黑树，然后不同的方式进行查找  

### fail-fast机制  

使用for-each遍历HashMap的时候，如果使用HashMap的remove方法，就会抛出ConcurrentModificationException  

```java
Map<String, Integer> map = new HashMap<>();  
map.put("1", 1);  
map.put("2", 2);  
map.put("3", 3);  
for (String s : map.keySet()) {  
    if (s.equals("2"))  
        map.remove("2");  
}
```

这就是所谓的fail-fast机制

在HashMap中有一个名为modCount的变量，它用来表示集合被修改的次数，修改指的是插入元素或者删除元素，在HashMap中每次的插入、删除都会对modCount进行自增操作  

当遍历HashMap的时候都会对modCount进行判断，若和原来的不一样说明集合被修改过了，然后就抛出异常  

```java
public Set<K> keySet() {  
    Set<K> ks = keySet;  
    if (ks == null) {  
        ks = new KeySet();  
        keySet = ks;  
    }  
    return ks;  
}  

final class KeySet extends AbstractSet<K> {  
    public final Iterator<K> iterator()     { return new KeyIterator(); }  
    // 省略部分代码  
}  

final class KeyIterator extends HashIterator implements Iterator<K> {  
    public final K next() { return nextNode().key; }  
}  

/*HashMap迭代器基类，子类有KeyIterator、ValueIterator等*/  
abstract class HashIterator {  
    Node<K,V> next;        //下一个节点  
    Node<K,V> current;     //当前节点  
    int expectedModCount;  //修改次数  
    int index;             //当前索引  
    //无参构造  
    HashIterator() {  
        expectedModCount = modCount;  
        Node<K,V>[] t = table;  
        current = next = null;  
        index = 0;  
        //找到第一个不为空的桶的索引  
        if (t != null && size > 0) {  
            do {} while (index < t.length && (next = t[index++]) == null);  
        }  
    }  
    //是否有下一个节点  
    public final boolean hasNext() {  
        return next != null;  
    }  
    //返回下一个节点  
    final Node<K,V> nextNode() {  
        Node<K,V>[] t;  
        Node<K,V> e = next;  
        if (modCount != expectedModCount)  
            throw new ConcurrentModificationException();//fail-fast  
        if (e == null)  
            throw new NoSuchElementException();  
        //当前的链表遍历完了就开始遍历下一个链表  
        if ((next = (current = e).next) == null && (t = table) != null) {  
            do {} while (index < t.length && (next = t[index++]) == null);  
        }  
        return e;  
    }  
    //删除元素  
    public final void remove() {  
        Node<K,V> p = current;  
        if (p == null)  
            throw new IllegalStateException();  
        if (modCount != expectedModCount)  
            throw new ConcurrentModificationException();  
        current = null;  
        K key = p.key;  
        removeNode(hash(key), key, null, false, false);//调用外部的removeNode  
        expectedModCount = modCount;  
    }  
}
```

相关代码如下  

```java
if (modCount != expectedModCount)  
    throw new ConcurrentModificationException();
```

可以看出如果modCount和原来的不相等，就会抛出错误  

那么如何在遍历的时候删除元素呢？其中最后两行代码如下  

```java
removeNode(hash(key), key, null, false, false);//调用外部的removeNode  
expectedModCount = modCount; 
```

意思是调用外部的remove方法删除元素后，把modCount赋值给expectedModCount，这样就不会抛出异常了  

```java
Map<String, Integer> map = new HashMap<>();  
map.put("1", 1);  
map.put("2", 2);  
map.put("3", 3);  
Iterator<String> iterator = map.keySet().iterator();  
while (iterator.hasNext()){  
    if (iterator.next().equals("2"))  
        iterator.remove();  
}
```

### hashCode eqauls ==  

"=="是用来比较变量的值是否相等。如果是基本类型，直接比较值；如果是对象类型，比较的是两个对象的引用，也就是地址。对象是放在堆中的，栈中存放的是对象的引用。"=="是对栈中值进行比较  

"equals" 是Object类的方法，在Object类中的equals方法，内部调用的就是"=="  

```java
public boolean equals(Object o) {
	return this == o;
}
```

**说明在Object类中"equals"和"=="是一回事**  

所以对一个类来说，如果类没有重写equals方法，那么对于该类来说 equals方法和==没有区别，都是比较对象的内存地址  有一些类中将equals进行了重写，例如String，所以不再是比较对象的内存地址   

一般重写了equals方法一定要重写hashCode方法  

**如果两个对象相同，那么它们的hashCode值一定相同；反过来，如果两个对象的hashCode值相同，不能说明这两个对象相同**  

所以在HashMap中判断时，首先会判断hashCode是否相同。如果不同，就说明不是同一个对象；如果相同，才会使用"=="方法进一步判断是不是同一个对象  