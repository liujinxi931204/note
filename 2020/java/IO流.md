### 流的概念和作用  
流(stream)是一组有顺序的、有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便值观的进行数据操作  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/03/1604394591834-1604394591921.png)  
### File类  
通过java.io.File类实现对文件的基本属性进行操作，包括文件属性读取、文件创建、文件删除、文件添加等。File是一个类，那么在使用的时候就需要创建对象，但是File类的实例是不可变的，也就是说，一旦创建，由File对象标识的抽象路径名将永远不会改变，也就是说，利用构造方法，指定路径名、文件名等来构造File类的对象，之后调用该对象的createNewFile()方法就可以创建出相应的文件  
#### File类的构造函数  
+ File(String pathname)  
通过将给定路径名字符串转化为抽象路径来创建一个新的File实例  
+ File(File parent,String child)  
根据parent抽象路径和child路径名字符串创建出一个新File实例  
+ File(String parent,String child)  
