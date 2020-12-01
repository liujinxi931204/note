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
})
```











