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
从java8 开始，无参函数的匿名内部类可以简写成如下