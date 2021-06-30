## Condition接口  

Lock接口中定义了newCondition方法，返回一个关联在Lock对象上的Condition对象。  

Condition接口的出现是为了扩展Synchronized中的wait、notify的机制。那么wait、notify有什么弊端呢？我们知道所用调用了wait的线程，都会被放在ObjectMonitor