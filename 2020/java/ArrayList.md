## 概述  
ArrayList是一种变长的集合类，基于定长数组实现。ArrayList允许空值和重复元素，当往ArrayList中添加的元素数量大于其底层数组容量时，其会通过扩容机制重新生成一个更大的数组  
ArrayList是线程不安全的，在并发环境下，多个线程同时操作ArrayList，会引发不可预知的错误  
## 源码分析  
### 属性分析  
```java
    /**
     * 默认初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 用于空对象的共享空数组
     * 用于创建指定长度为0的ArrayList的构造方法
     * java.util.ArrayList#ArrayList(int)
     * java.util.ArrayList#ArrayList(java.util.Collection)
     *
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 用于默认大小的空对象的共享空数组（跟上面的区分开来）
     * 用于创建默认长度为10的ArrayList的构造方法
     * java.util.ArrayList#ArrayList()
     *
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 存储内容的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList的长度
     */
    private int size;
    
    /**
     * 数组的最大长度
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```  
这里特别需要注意elementData.length是指数组的长度，size是指List的长度，这两个长度不是同一个概念  
### 构造方法  
```java
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // 初始容量大于0时，新建Object数组指向elementData
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // 初始容量等于0时，EMPTY_ELEMENTDATA数组指向elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        // 无参构造，elementData指向默认的空数组
        // 这也表明，elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA的话
        // 表示ArrayList还没有添加过任何元素
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        // collection转数组
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 数组长度不等于0
            // [2] c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                // [1]
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```  
构造方法做的事请并不复杂，目的都是初始化底层数组elementData。区别在于无参构造方法会将elementData初始化为一个空数组，插入元素时，扩容将会按照默认值重新初始化数组。而有参构造方法则会将elementData初始化为参数值大小(>=0)的数组。一般情况下，用默认的构造方法即可。倘若知道回想ArrayList插入多少元素，应该使用有参构造方法，按需分配，避免浪费  


