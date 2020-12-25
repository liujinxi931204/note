## ThreadLocal是什么  
ThreadLocal是JDK java.lang包中的一个用来实现不同线程的线程数据隔离的一个工具  
ThreadLocal这个类提供局部变量，这些变量与其他正常变量的不同之处在于，每一个访问该变量的线程在其内部都有一个独立的初始化的变量副本；ThreadLocal实例变量通常采用private static 在类中修饰  
只要ThreadLocal的变量能被访问，并且线程存活，那么每个线程都会持有ThreadLocal变量的副本。当一个线程结束的时候，它所持有的所有ThreadLocal相对应的实例副本都可被回收  
**ThreadLocal适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用(相同数据共享)，也就是变量在线程间隔离(不同的线程数据隔离)而在方法或类间共享的场景**  
    
**ThreadLocal的作用和同步机制有些相反：同步机制是为了保证多线程环境下数据的一致性；而ThreadLocal是保证了多线程环境下数据的独立性**  
## T