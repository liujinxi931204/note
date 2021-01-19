## Spring是什么  
Spring是分层的Java SE/EE 应full-stack轻量级开源框架，以IoC(Inverse of Control：控制反转)和AOP(Aspect Oriented Programming:面向切面编程)为内核  

提供了展现层Spring MVC和持久层Spring JDBCTemplate以及业务层事务管理等众多的企业级应用技术，还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的Java EE企业应用开源框架  
### Spring的优势  
1. **方便解耦，简化开发**  
通过Spring提供的IoC容器，可以将对象间的依赖关系交由Spring进行控制，避免硬编码所造成的过度耦合。用户也不必再为单例模式类、属性文件解析等这些很低层的需求编写代码，可以更专注于上层应用  
  
2. **AOP的编程支持**  
通过Spring的AOP功能，方便进行面向切面的编程，许多不容易用传统OOP实现的功能可以通过AOP实现  
  
3. **声明式事务的控制**  
可以将我们从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活的进行事务管理，提供开发效率和质量  
  
4. **方便程序的测试**  
可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情  
  
5. **方便集成各种优秀的框架**  
Spring对各种优秀的框架(Struts、Hibernate、Hession、Quartz等)的支持  
  
6. **降低Java EE api的使用难度**  
Spring对Java EE API(如JDBC、JavaMail、远程调用等)进行了薄薄的封装，使这些API的使用难度大为降低  
  
7. **Java源码是经典学习范例**  
Spring的源代码设计精妙、结构清晰、匠心独拥、，处处体现着大师对Java设计模式灵活运用以及对Java技术的高深造诣。它的源代码无疑是学习Java技术的最佳实践的范例  

## Spring体系结构  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/09/13/1599988281757-1599988281851.png) 




