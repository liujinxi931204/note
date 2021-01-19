### 流的概念和作用  
流(stream)是一组有顺序的、有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便值观的进行数据操作  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/03/1604394591834-1604394591921.png)  
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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/11/03/1604396729231-1604396729233.png)  
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
##### FileOutputStream输出字节流    
write(int b)  
write(byte[] b,int off,int len)  
##### 关闭流  
close()  
使用完流以后应该关闭相应的流，否则会占用一定的资源  
#### BufferedInputStream和BufferedOutputStream  
这两个是缓冲流，也是处理流，其构造方法需要接收节点流，即FileInputStram和FileOutputStream。这两个流可以使用缓冲区，不是每次和文件的操作都是实际的操作  
#### FileReader和FileWriter  
##### FileReader类  
FileReader以字符作为单位读取文本文件，能够以字符流的形式读取文件内容，除此之外，与FileInoutStream没有太多的区别  
##### FileWriter类  
FileWriter是文件字符输出流，主要将字符输出到指定的打开的文件中  

**FileWriter、FileReader与FileOutputStream、FileInputStream两个类的操作方法基本相同，若操作的文件不是文本文件，建议使用FileOutputStream、FileInputStream**  
#### BufferedReader和BufferedWriter  
##### BufferedReader  
能够为字符输入流提供缓冲区，可以提高许多IO的处理速度  
##### 常用方法  
read()  
read(char[],int off,int len)  
read(cahr[])  
返回一个正整数表示读入的字符数，如果返回-1，代表已经到达流的末尾  
readLine():返回该行内容的字符串，但不包括任何换行符，如果返回null，代表已经到达流的末尾  
close():关闭该流并释放资源  
#####BufferedWriter  
能够为其他的字符输出流提供缓冲区，提高效率  
##### 常用方法  
write(int c)：写入单个字符  
write(char[] cbuff,int off,int len)：将cbuff中从off开始到len结束的字符写入  
write(String str,int off,int len)：将str中off开头len结束的字符写入  
newLine()：换行  
flush()：将缓存的内容刷入文件  
close()：关闭流并释放资源  
#### InputStreamReader和OutputStreamWriter  
##### InputStreamReader  
将字节流转换为字符流，可以使用指定的charset读取字节并将其解码为字符  
##### 常用方法  
InputStreamReader(InputStream in)：创建一个使用默认字符集的InputStreamReader  
InputStreamReader(InputStream in, Charset cs):创建使用给定字符集的 InputStreamReader
InputStreamReader(InputStream in, CharsetDecoder dec):创建使用给定字符集解码器的 InputStreamReader。
InputStreamReader(InputStream in, String charsetName):创建使用指定字符集的 InputStreamReader。
close()；关闭流并释放资源  
read():读取单个字符  
read(char[] cr,int off,int len):将字符读入数组  
##### OutputStreamWriter  
将输出的字符流转化为字节流，可使用指定的charset将要写入流中的字符编码成字节  
##### 常用方法  
OutputStreamWriter(OutputStream out, String charsetName)：创建使用指定字符集的 OutputStreamWriter。  
OutputStreamWriter(OutputStream out, Charset cs)：创建使用给定字符集的 OutputStreamWriter。   
OutputStreamWriter(OutputStream out)：创建使用默认字符编码的 OutputStreamWriter。  
OutputStreamWriter(OutputStream out, CharsetEncoder enc)：创建使用给定字符集编码器的 OutputStreamWriter。  
fulsh():刷新该流的缓冲  
write(int c)：写入单个字符  
write(char[] cr,int off,int len):写入字符数组的某一部分  
write(String str,int off,int len):写入字符串的某一部分  
close():关闭流并释放资源  


**如果需要指定编码的格式，就需要使用InputStreamReader和OutputStreamWriter**  
**使用Bufferedread和BufferedWriter时，经常将FileRead和FileWriter放入其中，为的时提高效率**  
**如果提高效率最好还是使用BufferedReader、BufferedWriter对Reader和Writer及其子类进行包装**  



  




