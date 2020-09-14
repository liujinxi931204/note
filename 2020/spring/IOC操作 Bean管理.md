## 自动装配  
根据指定装配规则(属性名或者属性类型)，Spring自动将匹配的属性值进行注入  
  
自动装配的两种形式  
+ 根据属性名称来自动装配  
```java
配置文件applicationContext.xml
<!--
    bean标签的autowire属性实现自动装配
    autowire两个属性值：
        byName根据属性名进行注入，注入bean的id值和类属性名一致
-->

  <bean id="emp" class="com.sogou.spring5.Emp" autowire="byName"></bean>
  <bean id="dept" class="com.sogou.spring5.Dept"></bean>

Emp类实现


```
+ 根据属性类型来自动装配  