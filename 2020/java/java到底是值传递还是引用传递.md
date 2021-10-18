## Java到底是值传递还是引用传递

### 值传递和引用传递  

什么是值传递？什么是引用传递？

+ 值传递（pass by value）：是指在调用方法时将实参复制一份传递到方法中去，这样当方法对形参进行修改时不会影响到实参

+ 引用传递（pass by refernce）：是指在调用方法时将实参的地址直接传递到方法中去，那么在方法中对形参进行的修改，将影响到实参

总结起来，值传递和引用传递的区别有两点：

1. 调用方法时有没有对实参进行复制

2. 方法内对形参的修改会不会影响到实参

### 形参和实参

+ 形参：定义方法名和方法体时使用的参数，目的是用来接收调用该方法时传入的参数

+ 实参：在调用有参方法时传入的参数，方法名后面的括号中的参数通常被称为"实参"  

例如下面这个例子  

```java
public class Test{
    public static void main(String[] args){
        System.out.println("hello world");
    }
}
```

这个方法中，`args`就是`main`方法的形参，`"hello world"`则是实参。下面的这个例子会更加明显  

```java
public class Cmower {
    public static void main(String[] args) {
        Cmower cmower = new Cmower();
        cmower.sop("zhangsan");
    }

    public void sop(String name) {
        System.out.println("hello " + name);
    }
}
```

其中`name`是`sop`这个方法的形参，而`zhangsan`则是实参  

形参就好像实参与被调用方法之间的一个桥梁，否则调用者没法传递参数，被调用的方法无法接收参数。

### 基本数据类型是值传递  

java中的数据类型有两种，分别是基本数据类型和引用数据类型。其中基本数据类型的传递是值传递  

有这样一个例子

```java
public class test{
    public static void main(String[] args) {
        int x=1,y=2;
        swap(1,2);
        System.out.println("x: "+ x);
        System.out.println("y: "+ y);
    }

    public static void swap(int x,int y){
        int tmp=x;
        x=y;
        y=x;
    }
}
```

很明显方法`swap(int x,int y)`是实现两个参数进行交换的功能，但是实际上`main`方法中调用这个`swap`函数并没有达到两个数交换的目的。原因就在于，`int`是基本类型，是进行值传递的。也就是说，传递进`swap`方法的参数并不是实参本身，而是它们的拷贝。所以仅仅交换了实参的拷贝，而对实参本身并没有进行交换。

因此上述的输出如下  

```shell
x: 1
y: 2
```

### 引用数据类型是引用传递？  

看下面这个例子  

```java
public class Cmower {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Cmower cmower = new Cmower();
        cmower.setName("wanger");
        cmower.sop(cmower);
        System.out.println("main 中的 cmower " + cmower.getName());
    }

    public void sop(Cmower cmower) {
        cmower.setName("zhangsan");
        System.out.println("sop 中的 cmower " + cmower.getName());
    }
}
```

在 `main()` 方法中，我们通过 new 关键字创建了一个对象 cmower，并将其 name 属性设置为`"wanger"`；然后将实参 cmower 传递给 `sop()` 方法，在 `sop()` 方法中将形参 cmower 的 name 属性修改为`"zhangsan"`。输出结果是什么样子呢？

```shell
sop 中的 cmower zhangsan
main 中的 cmower zhangsan
```

做如下改动  

```java
public class Cmower {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Cmower cmower = new Cmower();
        cmower.setName("wanger");
        cmower.sop(cmower);
        System.out.println(cmower);
        System.out.println("main 中的 cmower " + cmower.getName());
    }

    public void sop(Cmower cmower) {
        cmower.setName("zhangsan");
        System.out.println(cmower);
        System.out.println("sop 中的 cmower " + cmower.getName());
    }
}
```

可以得到结果  

```shell
Cmower@1b6d3586
Cmower@1b6d3586
```

甚至会发现`sop`方法中的`cmower`和`main`方法中的`cmower`的内存地址都是相同的，这就更加说明引用类型是引用传递了  

这里看来，java的引用类型是引用传递

如果对上述代码做一点小小的改动，就会发现结果又不一样了  

```java
public class Cmower {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Cmower cmower = new Cmower();
        cmower.setName("wanger");
        cmower.sop(cmower);
        System.out.println(cmower);
    }

    public void sop(Cmower cmower) {
        cmower = new Comwer();//注意此处的变化
        cmower.setName("zhangsan");
        System.out.println(cmower);
    }
}
```

此时结果如下  

```shell
sop 中的 cmower zhangsan
Cmower@1b6d3586
main 中的 cmower wanger
Cmower@4554617c
```

此时会发现`sop`方法中的`comwer`和`main`方法中的`comwer`不是同一个对象了，并且`sop`方法对形参的修改对实参甚至没有生效。

可以发现区别仅仅是一句`comwer = new Cmower;`。所以这时还是引用传递吗？如果是引用传递，那么`main`方法中的`comwer`也会被改变

，所以可见java中的引用类型的传递不是引用传递而是值传递。

那么怎么解释上述的代码呢？在`main`方法中执行`Cmower cmower= new Cmower();`会在对空间中开辟一段内存，假设地址为0x123456。在执行`sop(cmower);`这一行指令时，会将这个地址拷贝一份，传递给`sop`这个函数。当没有`cmower = new Comwer();`这句改动时，形参`comwer`的地址也指向0x123456这段内存地址，因此`sop`函数中的修改就会反映到`main`方法中；反之，当有了`cmower = new Comwer();`这句改动时，形参`cmower`所指向的地址就被改变了，指向了一个新的地址，但肯定不会是0x123456了，所以`sop`方法中做出的修改就无法反映到`main`方法中了。

### 总结  

java中的参数传递是值传递，而不是引用传递。对于基本数据类型，传递的是原始数据的值的副本，对于引用数据类型，传递的是原始数据地址值的副本。
