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
Stream<String> paraStream=list.parallelStream();
```
### 通过Java.util.Arrays.stream(T[] array)方法用数组创建流  
```java
int[] array={1,2,5,7,9};
IntStream stream=Arrays.Stream(array);
```  
### 使用Stream的静态方法of()、iterate()、generate()  
```java
Stream<Integer> stream=Stream.of(1,2,3,4,5);

Stream<Integer> stream2=Stream.iterate(0,x->x+3).limit(4);
stream2.forEach(System.out::println);

Stream<Integer> stream3=Stream.generate(Math::random).limit(3);
Stream3.forEach(System.out::println);
```  
### stream和parallelStream的简单区分  
stream是顺序流，由主线程按顺序对流执行操作，而parallelStream是并行流，内部以多线程并行执行的方式对流进行操作，但前提是流中数据处理顺序没有要求  
除了直接创建并行流之外，还可以通过parallel()把顺序流转化成并行流  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606966971000-1606966971034.png)  
## Stream流使用  
### 遍历/匹配(forEach/find/match)  
Stream也是支持类似集合的遍历和匹配元素的，只是stream中的元素是以Optional类型存在的  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606978493081-1606978493087.png)  
```java
public class StreamTest{
    List<Integer> list=Arrays.asList(7,6,9,3,8,2,1);
//遍历符合条件的元素
    list.stream().filter(x->x>6).forEach(System.out::println);
//匹配第一个
    Optional<Integer> first=list.stream().filter(x->x>6).findFirst();
    System.out.println(first.get());
//匹配任意一个，可以使用并行流
    Optional<Integer> any=list.parallelStream().filter(x->x>6).findAny();
    System.out.println(any.get());
//是否包含符合特定条件的元素
    boolean b=list.stream().anyMatch(x->x>6);
    System.out.println(b);
}
```  
### 筛选  
筛选，是按照一定的规则校验流中的元素，将符合条件的元素提取到新的流中进行操作  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606980169070-1606980169074.png)  
```java
public class StreamTest {
	public static void main(String[] args) {
		List<Integer> list = Arrays.asList(6, 7, 3, 8, 1, 2, 9);
		Stream<Integer> stream = list.stream();
		stream.filter(x -> x > 7).forEach(System.out::println);
	}
}
```  
### 聚合  
max、min、count极大的方便了对集合、数组的数据统计工作  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606980809263-1606980809266.png)  
#### 获取长度最长的字符串  
```java
public void test3(){
        List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");
        Optional<String> max = list.stream().max((x1,x2)-> x1.length()-x2.length());
        System.out.println(max.get());
    }
```  
#### 获取数组中最小的元素  
```java
@Test
    public void test3(){
        List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);
        Optional<Integer> max = list.stream().min(Integer::compareTo);
        System.out.println(max.get());

    }
```  
#### 统计数组中大于6的元素的数量  
```java
    @Test
    public void test3(){
        List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);
        long count = list.stream().filter(x->x>6).count;
        System.out.println(count);

    }
```  
### 映射  
+ map:接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606982562306-1606982562309.png)  
+ flatMap:接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606985730953-1606985730957.png)  
#### 英文字符串数组的元素全部改为大写  
```java
    @Test
    public void  test4(){
        String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
        Stream<String> stream = Arrays.stream(strArr);
        stream.map(String::toUpperCase).forEach(System.out::println);
    }
``` 
#### 整数数组每个元素+3   
```java
    @Test
    public void  test4(){
        List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);
        intList.stream().map(x->x+3).forEach(System.out::println);
    }
```  
### 规约  
规约，也称缩减，是把一个流缩减成一个值，能够实现对集合求和、求乘积和求最值操作  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/03/1606988233272-1606988233276.png)  
```java
@Test
public void test5(){
                List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
		// 求和方式1
		Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
		// 求和方式2
		Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
		// 求和方式3
		Integer sum3 = list.stream().reduce(0, Integer::sum);
		
		// 求乘积
		Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

		// 求最大值方式1
		Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
		// 求最大值写法2
		Integer max2 = list.stream().reduce(1, Integer::max);

		System.out.println("list求和：" + sum.get() + "," + sum2.get() + "," + sum3);
		System.out.println("list求积：" + product.get());
		System.out.println("list求和：" + max.get() + "," + max2);
}
```
## 收集  
collect，可以说是内容最繁多、功能最丰富的部分了。从字面上理解，就是把一个流收集起来，最终可以是收集车工一个值也可以收集成一个新的集合  
**collect主要依赖java.util.stream.Collectors类内置的静态方法**  
### 归集  
因为流不存储数据，那么在流中的数据完成处理后，需要将流中的数据重新归集到新的集合里。toList、toSet、toMap比较常用  
#### 找出薪资大于8000的人的姓名和薪资  
```java

@Test
public void test2(){
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));
        personList.add(new Person("Owen", 9500, 25, "male", "New York"));
        personList.add(new Person("Alisa", 7900, 26, "female", "New York"));
        Map<String, Integer> collect = personList.stream().filter(x -> x.getSalary() > 8000).collect(Collectors.toMap(Person::getName, Person::getSalary));

        Set<Map.Entry<String, Integer>> entries = collect.entrySet();
        for(Map.Entry e :entries){
            System.out.println(e.getKey() + "" + e.getValue());
        }
        //也可以使用如下形式，更加简便
        //collect.forEach((k,v)-> System.out.println(k+"   "+v));
}
```

### 统计  
+ 计数  count  
+ 平均值  averageInt、averageDouble、averageLong  
+ 最值  maxBy、minBy  
+ 求和 summingInt、summingLong、summmingDouble  
+ 统计以上所有  summarizingInt、summarizingLong、summarizingDouble



