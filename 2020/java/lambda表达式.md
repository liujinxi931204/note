![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/01/1606821435116-1606821435140.png)  
## 概述  
java8 引入的lambda表达式的主要作用就是简化部分的写法  
能够使用lambda表达式的一个重要依据是必须有相应的**函数接口**。所谓函数接口，是指内部有且仅有一个抽象方法的接口  
lambda表达式的另一个依据是**类型推断机制**。在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显式指名  
## 常见用法  
### 无参函数的简写  
无参函数就是没有参数的函数，例如Runnable接口的run()方法，其定义如下  
```java
@FunctionalInterface
public interface Runnable{
    public adstract void run();
}
```  
在没有使用lambda的时候，可以使用匿名内部类的形式  
```java
new Thread(new Runnable(){
    @Override
    public void run(){
        System.out.println("hello");
        System.out.println("Jimmy");
    }
}).start();
```
从java8 开始，无参函数的匿名内部类可以简写成如下方式：  
```java
()->{
    执行语句
}
```
这样接口名和函数名就可以省略掉了。那么，上面的示例可以简写成：  
```java
new Thread(()->{
    System.out.println("hello");
    System.out.println("Jimmy")；
}).start();
```  
如果执行的语句只有一条的时候，还可以对代码块进行简写  
```java
()->执行语句
```
所以，如果上面的例子中只有一条执行语句的时候，还可以简写  
```java
new Thread(()->System.out.println("hello")).start();
```
### 单参数函数的简写  
单参数函数是指只有一个参数的函数。例如，View内部的接口OnClickListener的方法，onClick(View v)，其定义如下  
```java
public interface OnClickListener{
    void onClick(View v);
}
```  
不使用lambda表达式可以使用如下的方式  
```java
view.setOnClickListener(new OnClickListener(){
    @override
    public void onClick(View v){
       v.setVisibility(View.GONE);
   }
});
```
从java8 开始，单参数的匿名内部类可以简写成如下  
```java
([类名] 变量名)->{
    执行语句
}
```
其中类名是可以省略的，因为lambda表达式是可以自己推断出类型的。那么上面的例子可以简写成如下  
```java
view.setOnClickListener((View v)->{
    v.setVisibility(View.GONE);
});

view.setOnClickListener((v)->{
    v.setVisibility(View.GONE);
});
```
单参数函数甚至可以把括号去掉，官方也更建议使用这种方式  
```java
变量名->{
    执行语句;
}
```
上述例子可以写成如下形式  
```java
view.setOnClickListener(v->{
    v.setVisibility(View.GONE);
});
```  
当只有一句执行语句的时候，依然可以对代码块进行简化  
```java
变量名->执行语句 
```  
那么上面的例子还可以简化成如下格式  
```java
view.setOnClickListener(v->v.setVisibility(View.GONE));
```
### 多参数函数的简写  
多参数函数是指具有两个及以上参数的函数。例如Comparator接口的compare(T o1,T o2)方法就具有两个参数，其定义如下  
```java
@FunctionalInterface
public interface Comparor<T> {
    int compare(T o1,T o2)；
}
```  
在不适用Lambda表达式的时候，当我们对一个集合进行排序时，通常会这么写  
```java
ArrayList<Integer> list=Array.asList(1,2,3);
Collections.sort(list,new Compartor<Integet>{
    @Override
    public int compare(Integer o1,Integer o2){
        return o1.compareTo(o2);
    }
})
```  
多参数的匿名内部类可以使用Lambda表达式简写成如下形式  
```java
([类名1]变量名1，[类名2]变量名2)->{
    执行语句；
}
```
同样类名可以省略，那么上面的例子可以写成  
```java
Collections.sort(list,(Integer o1,Integer o2)->{
    o1.compareTo(o2);    
});  
Collections.sort(list,(o1,o2)->{
    o1.compareTo(o2);
});
```  
当只有一条执行语句的时候，依然可以对代码块进行简化  
```java
([类名1] 变量名1，[类名2] 变量名2)->z执行语句
```  
此时类名可以省略，但是括号不能省略。如果这条语句需要返回值，那么return也可以省略。那么上面的例子就可以简化为  
```java
Collections.sort(list,(o1,o2)->o1.compareTo(o2));
```
## Java内置四大核心函数式接口  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/02/1606874161734-1606874161736.png)  
### 消费型接口  
Consumer<T> 消费型接口  
```java
void accept(T t);
```
```java
public void hello(String str,Consumer<String> con){
    con.accept(str);
}

@Test
public void test1(){
    hello("张三",m->System.out.println("你好"+m));
}
```  
### 供给型接口  
Supplier<T> 供给型接口  
```java
T get();
```
```java
//需求：产生指定个数的整数，并放入集合
public List<Integer> getNumList(int num,Supplier<Integer> sup){
    List<Integer> list=new ArrayList<>();
    for(int i=0;i<num;i++){
        Integer n=sup.get();
        list.add(n)；
    }
    return List;
}

@Test
public void test2(){
    List<Integer> numList=getNumList(1,()->(int)(Math.random()*100));
    for(Integer num:nunList){
        System.out.println(num);
    }
}
```  
### 函数型接口  
Function<T,R> 函数型接口  
```java
R apply(T t);
```
```java
public String setHandler(String str,Function<String,String> func){
    return func.apply(str);
}

@Test
public void test3(){
    String newStr1=setHander("ttt 这是一个函数接口",str->str.trim());
    System.out.println(newStr1);
}
```  
### 断定型接口  
Predicate<T> 断定型接口  
```java
boolean test(T t);
```
```java
public List<String> filterStr(List<String> str,Predicate<String> pre){
    List<String> strList=new ArrayList<>();
    for(String str:list){
        if(pre.test(str)){
             strList.add(str);
        }
    }
    return strList;
}
@Test
public void test4(){
    List<String> list=Arrays.asList("hello","java","Lambda","www","ok");
    List<String> strList=filterStr(list,str->str.length()>3);
    for(String str:strList){
        System.out.println(str);
    }
}
```
## 方法引用  
方法引用也是一个语法糖，可以用来简化开发  
在使用Lambda表达式的时候，如果"->"的右边要执行的表达式只是调用一个类已有的方法，那么就可以用方法引用来替代Lambda表达式  
+ 引用静态方法  
+ 引用对象方法  
+ 以用类方法  
+ 引用构造方法  
### 引用静态方法  
当要执行的表达式是调用某个类的静态方法，并且这个静态方法的参数列表和接口里抽象函数的参数列表一一对应，可以采用引用静态方法的格式  
假如Lambda表达式符合如下格式  
```java
([变量1，变量2，...])->类名.静态方法名([变量1，变量2...])
```
可以简化成如下格式  
```java
类名::静态方法名 
```  
注意这里静态方法名后面不需要加括号，也不用加参数，因为编译器可以推断出类型  
首先创建一个工具类，代码如下  
```java
public class Utils{
    public static int compare(Integer o1,Integer o2){
        return o1.compareTo(o2);
    }
}
```  
注意这里的compare()函数的参数和Comparable接口的compare()函数的参数是一一对应的。然后一般的Lambda表达式可以写成如下形式  
```java
Collectoins.sort(list,(o1,o2)->o1.compareTo(o2));
```  
如果采用引用的方式可以写成如下形式  
```java
Collections.sort(list,Utils::compare);
```  
### 引用对象的方法  
当要执行的表达式是调用某个对象的方法，并且这个方法的参数列表和接口里抽象函数的参数一一对应时，就可以采用引用对象的方法的格式  
假如Lambda表达式符合如下格式  
```java
([变量1,变量2,...])->对象引用.方法名([变量1,变量2,...])
```
可以简写成如下形式  
```java
对象引用::方法名
```  
例如创建如下类  
```java
public class myClass{
    public int compare(Integer o1,Integer o2){
        return o1.compareTo(o2);
    }
}
```  
当我们创建一个该类的对象，并在Lambda表达式中使用该对象的方法时，一般可以这么写  
```java
myClass myclass=new myClass();
Collections.sort(list,(o1,o2)->myclass.compare(o1,o2));
```





