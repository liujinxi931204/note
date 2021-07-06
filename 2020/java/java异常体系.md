## java异常体系  

**引入异常处理机制可以解决的问题有**  

1. 使用方法的返回值来表示异常情况有局限，需要无穷列举所有的异常情况  

2. 异常流程代码和正常流程代码混合在一起，代码的可读性和可维护性都不高  

3. 随着系统规模的不断扩大，代码很难维护，特别是系统可扩展性差  

Java基于面向对象的思想提出了解决方案  

1. 把不同类型的异常情况使用不同的类来表示，不同的异常类型有共同父类  

2. 分离异常流程代码和正确流程代码  

3. 规范了异常处理机制，能处理的就将其捕获处理；不能处理的，就交给其调用者来处理  

### java异常体系  

![java异常体系](https://gitee.com/liujinxi931204/typoraImage/raw/master/img/java%E5%BC%82%E5%B8%B8%E4%BD%93%E7%B3%BB.png)  

**Thorwable**：在java体系中，Throwable是所有异常和错误的父类  

**Error**：表示错误，一般指JVM相关的不可修复的错误，例如系统崩溃、内存溢出、JVM内部错误等，由JVM抛出，一般情况下不需要处理，几乎其所有的子类都是以"Error"做为类名的后缀的。例如StackOverflowError，当应用程序递归太深而发生内存溢出，就会抛出该错误  

**Exception**：表示异常，指程序中出现不正常的情况，异常一般都是需要程序员来处理的（捕获或者抛出）；几乎其所有的子类都是以"Exception"作为类名的后缀的。  

Exception可以分为运行时异常（RuntimeException）和非运行时异常  

**RuntimeException**：该类异常属于程序运行时异常，也就是有程序员自身的问题导致产生的异常。例如NullPointerException空指针异常。该类异常在语法上不强制程序员必须处理，即使不处理这样的异常也不会出现语法错误  

**非运行时异常**：该类异常属于程序外部的问题引起的异常，也就是程序运行时由于某些外部问题导致产生的异常。例如文件不存在导致FileNotFoundException。该类异常在语法上强制程序员必须处理，如果不进行处理会出现语法错误。  

非运行时异常又被称为检查异常（CheckedException），Error和RuntimeException又被称为非检查异常（UncheckedException）  

### 异常的处理方式  

如果出现异常，会立刻中断正在运行的程序，所以必须要处理，而处理的方式有两种  

1. **throws**：当前方法不处理，而是声明抛出，由该方法的调用者来处理  
2. **try-catch**：在当前方法中使用try-catch的语句块来处理异常

#### try-catch捕获异常  

语法如下  

```java
/**
* 捕获单个异常
*/
try{
    可能出现异常的代码
}catech（异常类型 e）{
    /**
    * 异常处理
    * 记录日志、打印异常信息、继续抛出异常
    */
}

/**
* 捕获多个异常
*/
try{
   可能出现异常的代码
}catch（异常类型 a）{
    /**
    * 异常处理
    * 记录日志、打印异常信息、继续抛出异常
    */
}catch（异常类型 b）{
    /**
    * 异常处理
    * 记录日志、打印异常信息、继续抛出异常
    */
}
//注意：try、catch都不能单独使用，必须连用
```

注意：  

1. 一个catch语句，只能捕获一种类型的异常，如果需要捕获多种异常类型，需要使用多个catch语句  

2. try-catch中的代码中的异常只能被一个catch语句捕获，不能被多个catch语句捕获  

3. 在有多个catch语句的代码中出现异常，会从上到下匹配catch语句，所以多个catch语句匹配的异常类型应该按照从子类到父类的顺序定义  

4. 一旦匹配上其中一个catch语句，便不会匹配剩余的catch，而是会跳出try-catch，执行之后的代码  

### 抛出异常  

抛出异常的方式有两种  

1. **throw**：用于方法内部，用于给调用者返回一个异常对象，和return一样会结束当前方法  

2. **throws**：运行于方法声明之上，定义于方法参数之后，表示当前方法不处理异常，而是提醒该方法的调用者来处理可能抛出的异常（一个或者多个）  

#### throw语句  

运用于方法的内部，抛出一个具体的异常对象，终止方法的执行  

```java
throw new 异常类型("异常信息");
```

一般的，当一个方法出现异常情况而不知道应该返回什么时，此时就可以返回一个错误。return是返回一个正常值，throw是返回一个错误。例如String类的charAt方法  

```java
public char charAt (int index){
    if(index<0 || (index >= value.lenght)){
        throw new StringIndexOutofBoundException;
    }
    return value(index);
}
```

#### throws语句  

如果每一个方法都放弃异常处理直接通过throws声明抛出，最后异常会抛到main方法，如果此时main方法还不处理，会继续抛出给JVM，JVM底层的处理机制就是打印异常的跟踪栈信息；runtime异常，默认就是这种处理方式  

方法重写(Overrides)中的throws  

**一同**：方法的签名必须相同  

**两小**：

1. 子类方法的返回值类型和父类方法的返回值类型相同或者是其子类  

2. 子类方法不能声明抛出新的异常  

**一大**：子类方法的访问权限必须大于等于父类方法的访问权限  

### 自定义异常  

自定义异常的方式  

1. **受检查异常**：自定义一个受检查的异常类需要继承java.lang.Exception  

2. **运行时异常**：自定义一个运行时的异常类需要继承java.lang.RuntimeException  

一般开发中，自定义的异常都是运行时异常  

### finally代码块  

finally语句块表示无论如何（也包括异常发生时）都会最终执行的代码块  

#### finally的两种用法  

1. **try-finally**：此时没有catch来捕获异常，因为此时根据应用场景会抛出异常，程序员自己不处理  

   ```java
   try{
       //可能出现异常的语句
   }finally{
       //始终都会执行的语句
   }
   ```

2. **try-catch-finally**：程序员需要自己处理异常  

   ```java
   try{
       //可能出现异常的语句
   }catch(异常类型 e){
       //捕获异常，异常处理
   }finally{
       //始终否会执行的语句
   }
   ```

需要注意的是：finally不能单独使用  

#### finally不执行的情况  

当只有在try或者catch调用退出JVM的相关方法，此时finally才不会执行，否则finally修饰的代码块永远会执行  

```java
public class ExceptionDemo{
    public static void main(String[] args){
        try{
            int result = 10/0;
            System.out.println("10/2="+result);
        }catch(ArithmeticException e){
            System.out.println("异常信息： "+e.getMessage());
            System.exit(0);//退出JVM
        }finally{
            System.out.println("关闭资源");
        }
    }
}
```

在上述的例子中，用于在catch中调用了System.exit(0)，此时会退出JVM，所以finally中的代码就不会执行  

#### finally和return  

如果return同时存在于finally之内和之外，那么永远返回finally之内的结果  

```java
public class ExceptionDemo{
    public static void main(String[] args){
        //结果是14
        System.out.println(test1());
    }
    
    public int test1(){
        try{
            return 12;//不会被返回
        }finally{
            return 14;//会被返回,finally中的结果始终都会被返回
        }
    }
}
```

如果return仅仅存在于finally之外，那么将返回这个值  

```java
public class ExceptionDemo{
    public static void main(String[] args){
        //结果是13
        System.out.println(test1());
    }
    
    public int test1(){
        int a =13;
        try{
            return a;//会被返回
        }finally{
            a++;
            System.out.println(a);//打印14，但是不会被返回
        }
    }
}
```

首先finally肯定是会被执行的，所以a++之后就变成了14，然后打印了14；但是值为14的变量没有被返回；然后执行return a;这里的a在方法执行之处就已经确认了是13，故返回的值是13  

#### 异常处理原则  

1. 异常只能用于非正常的情况，try-catch的存在也会影响性能，尽量缩小try-catch的范围  

2. 需要为异常提供说明文档，可以参考java doc，如果自定义了异常或者一个方法抛出了异常，应该在文档中注释中详细说明  

3. 尽可能避免异常的出现，例如NullPointerException  

4. 异常的粒度很重要，应该为一个基本操作定义一个try-catch块，切忌将几百行代码放到一个try-catch中  

5. 自定义异常尽量使用RuntimeException类型的，并且要尽量避开已存在的异常  


