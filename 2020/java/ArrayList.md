## 概述  
ArrayList是一种变长的集合类，基于定长数组实现。ArrayList允许空值和重复元素，当往ArrayList中添加的元素数量大于其底层数组容量时，其会通过扩容机制重新生成一个更大的数组  
ArrayList是线程不安全的，在并发环境下，多个线程同时操作ArrayList，会引发不可预知的错误  
## 源码分析  
### 属性分析  
```java
/**
 * 默认初始化容量
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 如果自定义容量为0，则会默认用它来初始化ArrayList。或者用于空数组替换。
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 如果没有自定义容量，则会使用它来初始化ArrayList。或者用于空数组比对。
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 这就是ArrayList底层用到的数组

 * 非私有，以简化嵌套类访问
 * transient 在已经实现序列化的类中，不允许某变量序列化
 */
transient Object[] elementData;

/**
 * 实际ArrayList集合大小
 */
private int size;

/**
 * 可分配的最大容量
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```