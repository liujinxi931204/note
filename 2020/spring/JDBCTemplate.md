## 什么是JDBCTemplate  
+ Spring框架对JDBC进行封装，使用JDBCTemplate方便实现对数据库的操作  
+ 准备工作  
1. 准备jar包，引入依赖  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/09/20/1600611409859-1600611409920.png)  
2. 在Spring配置文件中配置数据库连接池（也可以使用配置jdbc.properties，然后读取到Spring的配置文件中）  
```java
<!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql://10.160.58.128/user_db" />
        <property name="username" value="root" />
        <property name="password" value="123465" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    </bean>
```  
3. 配置JDBCTemplate对象，注入dataSource对象  
