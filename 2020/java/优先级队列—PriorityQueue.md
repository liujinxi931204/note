## PriorityQueue  

PriorityQueue也被称为优先队列，在java中，通过优先队列实现了数据结构中的堆的结构，默认情况下实现的是一个小顶堆，也就是每次从堆顶取出的元素是整个堆中最小的元素。当然也可以通过传入一个比较器comparator从而实现一个大顶堆。我们知道堆是一个完全二叉树，所以可以利用数组来实现这个完全二叉树，这情况下父子节点的关系就可以利用数组的下标来表示了。PriorityQueue也正是利用数组来实现堆这个数据结构的  

![PriorityQueue](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/PriorityQueue.png)  

使用数组实现完全二叉树一定要注意节点之间的关系  
$$
leftNo = parentNo*2+1
$$

$$
rightNo = parentNo*2+2
$$

$$
parentNo=(leftNo-1)/2=(right-1)/2
$$

通过上面三个公式，可以很轻松的计算出某个节点的父节点以及子节点的下标，PriorityQueue的peek、element操作是常数时间，add、offer、remove、poll操作的时间复杂度是log(N)  

## 方法剖析  

### 基本属性  

```java
//数组的默认初始化长度
private static final int DEFAULT_INITIAL_CAPACITY = 11;
//存储元素的数组，可以看到是一个Object的数组
transient Object[] queue; // non-private to simplify nested class access
//元素的个数
int size;
//比较器，在定义大顶堆或者自定义的时候需要用到
private final Comparator<? super E> comparator;
//数组中元素被修改的次数，有这个属性表示优先队列是快速失败的
transient int modCount;    
```

### 构造方法  

PriorityQueue的构造函数有7个，这里只介绍了其中的三个

```java
//没有参数的构造函数
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}
//指定数组大小的构造函数
public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}
//指定数组大小和比较器的构造函数
public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
```

无参的构造函数，内部会将数组的初始化长度设置为DEFAULT_INITIAL_CAPACITY；指定数组大小的构造函数则会直接使用传入的长度。这两个方法都会在内部调用PriorityQueue(int initialCapacity,Comparator<? super E> comparator)这个构造函数。在PriorityQueue(int initialCapacity,Comparator<? super E> comparator)构造函数的内部会创建一个指定长度的Object数组并且设置comparator比较器。

可以看到优先队列没有使用懒加载的方式创建数组

### 添加元素

#### add方法  

```java
//在队列的尾部添加一个元素，然后在方法的内部又调用了offer方法
public boolean add(E e) {
        return offer(e);
}
```

可以看到add方法其实没有做什么，仅仅是在内部简单的调用了一下offer方法，下来看一下offer方法

#### offer方法  

```java
public boolean offer(E e) {
    //如果是null，直接抛出异常，说明priority queue不能添加null元素
    if (e == null)
        throw new NullPointerException();
    //数组中修改元素的次数加一
    modCount++;
    int i = size;
    //先检查数组中元素的数量，如果大于等于数组的长度，数组扩容
    if (i >= queue.length)
        grow(i + 1);
    //堆节点调整，堆化
    siftUp(i, e);
    //数组中的元素数量加1
    size = i + 1;
    return true;
}
```

  接下来看一下扩容的方法grow

```java
private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%
    //如果数组的长度小于64，扩容后的长度为原来的2倍;如果数组的长度大于等于64，扩容后的长度为原来的1.5倍
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                     (oldCapacity + 2) :
                                     (oldCapacity >> 1));
    // overflow-conscious code
    //如果队列的长度非常巨大，有特殊的处理逻辑
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //将原来数组中的内容拷贝到新的数组中
    queue = Arrays.copyOf(queue, newCapacity);
}
```

可以看到如果数组的长度小于64的时候，每次会扩容2倍；如果数组的长度大于等于64，那么每次扩容1.5倍。然后将旧数组的中元素全部拷贝到新的数组中，完成扩容。

这里以小顶堆为例，扩容完成以后，就需要对数组中的元素进行调整，以满足小顶堆的定义

#### siftUp方法  

```java
//传入的k是数组中添加元素的下标，x是需要添加的元素
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x, queue, comparator);
    else
        siftUpComparable(k, x, queue);
}
```

siftUp方法的内部又根据是否有比较器compartor来调用不同的方法，如果存在比较器，则调用siftUpUsingComparator方法，否则调用siftUpComparable方法。因为小顶堆是优先队列的默认实现，所以没有比较器，这里就直接进入siftUpComparable方法  

#### siftUpComparable方法  

```java
private static <T> void siftUpComparable(int k, T x, Object[] es) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        //父节点的小标
        int parent = (k - 1) >>> 1;
        Object e = es[parent];
        //如果要添加的元素比父节点的元素大，不需要对节点进行调整
        if (key.compareTo((T) e) >= 0)
            break;
        //如果要添加的元素比父节点的小，需要对节点进行调整
        //与父节点交换元素
        es[k] = e;
        //现在插入位置变为了父节点的位置，然后再与新的父节点比较
        k = parent;
    }
    //最后找到应该插入的位置，插入元素
    es[k] = key;
}
```

可以用下图来表示一下上述堆化的过程

![堆化](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%A0%86%E5%8C%96.png)

#### 总结  

可以看出添加元素的逻辑比较简单，主要是将元素添加到数组的末尾，这也符合队列的定义。然后将队尾的元素和其父节点的元素进行比较。如果元素比父节点的元素大，那么添加流程就结束了；如果元素比父节点的元素要小，那么就需要进行堆化调整，最后整个还是符合小顶堆的定义

### 获取堆顶的元素  

获取堆顶元素使用的是peek方法，来看一下peek方法  

#### peek方法  

```java
public E peek() {
    return (E) queue[0];
}
```

可以看到这个方法比较简单，主要就是获取数组下标位置为0的元素

### 删除元素  

#### poll方法  

```java
public E poll() {
    final Object[] es;
    final E result;
    //result在数组下标位置0的元素不为空的情况下，赋值为下标为0位置的元素
    if ((result = (E) ((es = queue)[0])) != null) {
        modCount++;
        final int n;
        //x赋值为数组的最后一个元素
        final E x = (E) es[(n = --size)];
        //将最后一个位置的元素情况
        es[n] = null;
        if (n > 0) {
            //堆化调整，使得整个堆赋值小顶堆的定义
            final Comparator<? super E> cmp;
            if ((cmp = comparator) == null)
                siftDownComparable(0, x, es, n);
            else
                siftDownUsingComparator(0, x, es, n, cmp);
        }
    }
    //返回已经被赋值的result
    return result;
}
```

poll方法的作用是删除堆顶元素并返回，之后还要调整整个堆，使得其符合堆的定义。在这个方法中，首先将数组下标为0的元素赋值给result，然后将数组最后一个位置的元素赋值给x并且情况最后一个位置的元素。最后对整个堆进行调整，使得其满足堆的定义。siftDownComparable和siftDownUsingComparator的作用是对整个堆进行调整，区别在于后者需要传入一个自定义的比较器。这里看一下siftDownComparable方法  

#### siftDownComparable方法  

```java
private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
    // assert n > 0;
    Comparable<? super T> key = (Comparable<? super T>)x;
    //找到最后一个非叶子节点
    int half = n >>> 1;           // loop while a non-leaf
    while (k < half) {
        //child是k的左子节点
        int child = (k << 1) + 1; // assume left child is least
        Object c = es[child];
        //right是k的右子节点
        int right = child + 1;
        //选择k的左子节点和右子节点中较小的一个，与k进行交换
        if (right < n &&
            ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
            c = es[child = right];
        if (key.compareTo((T) c) <= 0)
            break;
        es[k] = c;
        //k移动到左子节点和右子节点中较小的那个节点，重复上述的过程
        k = child;
    }
    //找到正确的位置，放入原来数组中最后一个位置的元素
    es[k] = key;
}
```

上述的过程可以用下图来表示

![删除堆顶元素](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/%E5%88%A0%E9%99%A4%E5%A0%86%E9%A1%B6%E5%85%83%E7%B4%A0.png)  

#### 总结  

poll方法的主要作用就是删除栈顶元素并返回，而在这个方法的内部首先会记录这个栈顶元素和数组最后位置的元素，然后将最后这个位置的元素清空。然后从堆顶元素开始，寻找其左孩子和右孩子中最小的节点替代该父节点的元素，然后继续从较小的这个孩子节点继续循环，直到到达最后一个非子叶节点，结束循环

### 总结  

相比普通的数组，优先队列的插入、删除操作的时间复杂度都是O(logn)，不需要频繁的移动元素，效率较高。每次的添加元素和删除元素都需要自动调整堆，使得其符合堆的定义。
