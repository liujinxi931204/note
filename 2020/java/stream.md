## Stream概述  
Stream将要处理的元素集合看作一种流，在流的过程中，借助Stream API对流中的元素进行操作，比如筛选、排序、聚合等  
Stream可以由数组或集合创建，对流的操作分为两种  
+ 中间操作，每次返回一个新的流，可以有多个  
+ 终端操作，每个流只能进行一次终端操作，终端操作结束后流无法再次使用。终端操作会产生一个新的集合或值  
另外，Stream有几个特性  
+ stream不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果  
+ stream不会改变数据源，通常情况下会产生一个新的集合或一个值  
+ stream具体延迟执行特性，只有调用终端操作时，中间操作才会执行  
## Stream的创建  
### 通过Java.util.Collection.stream()方法用集合创建  
```java
List<String> list=Arrays.asList("a","b","c");
//创建一个顺序流  
Stream<String> stream=list.stream();
//创建一个并行流  
Stream<String> pa
```