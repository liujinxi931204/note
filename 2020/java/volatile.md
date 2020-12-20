volatile可以看作是轻量级的synchornized，它只保证共享变量的可见性。在线程A修改被volatile修饰的变量后，线程B能够读取到正确的值。java在多线程中操作共享变量的过程中，会存在指定重排序与共享变量工作内存缓存的问题  
## java内存模型  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/20/1608435333675-1608435333684.png)  
