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
这里特别需要注意elementData.lengt