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
parent指定路径(父目录),也可以是File类对象，child中也可以加入路径层级  
#### File类常用方法  
+ createNewFile(): 创建文件  
+ delete(): 删除文件，如果删除的是文件夹，则文件必须为空；如果要删除一个非空的文件夹，需要删除该文件夹下的所有文件和文件夹  
+ mkdir()和mkdirs(): 创建文件夹，mkdir()要求父目录必须存在；mkdirs()不要求父目录必须存在  
+ list()：如果路径名不标识目录，则返回null；否则将返回一个字符串数组，表示该目录下所有文件名  
+ listFiles(): 返回目录下所有指定文件  
+ exists(): 判断文件或文件夹是否存在    
### IO流的分类  
根据处理数据类型的不同分为：字符流和字节流  
根据数据流向方向的不同分为：输入流和输出流  
从流的角色划分：节点流和处理流  
1. 节点流：直接连接数据源的流，可以直接向数据源(特定IO设备)读写数据  
2. 处理流：通过构造方法接收一个节点流，对节点流使用装饰着模式增加更多的功能，处理流必须依赖于一个节点流，因为只有节点流最终可以将数据流输入输出到IO设备中  

### IO流的4大基类  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/11/03/1604396729231-1604396729233.png)  
#### FileInputStream和FileOutputStream  
FileInputStream和FileOutputStream是两个常用的文件字节输入输出流，主要用于对文件以字节的方式来处理、如音乐、视频、图片等  
##### FileInputStream读取数据流字节  
read()  
read(byte[] b)  
read(byte[] b,int off,int len)  
##### 参数 
b: 存储读入数据的缓冲区  
off: 数据的初始偏移量  
len: 读取的最大的字节数  
##### 返回  
如果因为已经到达流的末尾而不再有数据可用，则返回-1  

  




