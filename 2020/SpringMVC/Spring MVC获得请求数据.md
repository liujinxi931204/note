### 获得请求参数  
客户端请求参数的格式为：name=value&name=value&...  
服务器端要获得请求的参数，有时还需要进行数据的封装，Spring MVC可以接受如下类型的参数  
+ 基本类型参数  
+ POJO类型参数(简单的java bean)  
+ 数组类型参数  
+ 集合类型参数  
### 获得基本类型参数  
Controller中的业务方法的参数名称与请求参数的name一致，参数值会自动映射匹配  
```java
@RequestMapping("/quick")
@RequestBody//代表不进行页面跳转，直接在页面回写数据
public void quickMethod(String name,int age){
    System.out.println(name);
    System.out.println(age);
}
```
在浏览器输入以下地址:http://localhost:8080/quick?name=xxx&age=18  
可以得到name=xxx age=18的请求参数  
### 获得POJO类型参数  
Controller中的业务方法的POJO参数的属性名与请求参数的name一致，参数值就会自动映射匹配  
```java
public class User {
    private String name;
    prinvate int age;

    //get、set方法
}


@RequestMapping("/quick")
@RequestBody
public void quickMethod(User user) throw IOException{
    Systemt.out.println(user);
}
```  
在浏览器输入以下地址：http://localhost:8080/quick?name=xxx&age=18  
就可以在方法quickMethod中将name和age封装进user实例中，从而获得一个user对象  
### 获得数组类型参数  
Controller中的业务方法数组名称与请求参数的name一致，参数值会自动映射匹配  
```java
@RequestMapping("/quick")
@RequestBody
public void quickMethod(String[] strs) throw IOException{
    Systeml.out.println(Arrays.asList(strs));
}
```
在浏览器输入以下地址：http://localhost:8080/quick?strs=111&strs=222&strs=333  
就可以在方法quickMethod中得到strs数组，数组的内容是strs=[111,222,333]  
### 获得集合类型参数  
+ 获得集合参数时，要将集合参数包装到一个POJO中才可以  
```java
public class User{
    private int id;
    private int name;
    //get、set方法
}

public class Vo {
    private List<User> userList;
    //get、set、toString方法
}

@RequestMapping("/quick")
@ResquestBody
public void quickMethod(Vo vo){
    System.out.println(vo);
}
```
```jsp
<form action="success.jsp" method="post">
<%--        表明是第几个User对象的userId、userName--%>
        <input type="text" name="userList[0].id"><br>
        <input type="text" name="userList[0].name"><br>
        <input type="text" name="userList[1].id"><br>
        <input type="text" name="userList[1].name"><br>
        <input type="text" name="userList[2].id"><br>
        <input type="text" name="userList[2].name"><br>
        <input type="submit" name="submit" >
    </form>
```  
在页面的表单中提交这些数据，就可以获得Vo{userList=[User{id=111,name="xxx"},User{id=222,name="yyy"}]}这样的集合  
+ 当使用ajax提交时，可以指定contextType为json形式，那么在方法参数位置使用@ReuqestBody可以直接接收集合而无需使用POJO进行包装  
示例如下：  
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
<%--    引入jQuery--%>
    <script src="${pageContext.request.contextPath}/js/jquery-2.1.3.min.js"></script>
    <script>
        var userList=new Array();
        userList.push({id:"1",name:"xxx"});
        userList.push({id:"2",name:"yyy"});
    //    发送ajax请求
        $.ajax({
            type:"POST",
            url:"${pageContext.request.contextPath}/quick",
            data:JSON.stringify(userList),
            contentType:"application/json;charset=utf-8"
        });
    </script>
</head>
<body>

</body>
</html>

```  
在页面一开始的时候发送ajax请求，请求中设置userList  
```java
package com.sogou.controller;

import com.sogou.service.User;
import com.sogou.service.Vo;
import org.springframework.core.StandardReflectionParameterNameDiscoverer;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.PriorityQueue;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-22 0:04
 **/

@Controller
public class UserController {

    @RequestMapping("/quick")
    @ResponseBody
    public void quickMethod(@RequestBody List<User> userList){
        System.out.println(userList);
    }
}

```  
这时可以在方法中使用集合类型来接收传递过来的集合类型的参数  
#### 开启允许访问静态资源  
在Spring MVC中访问静态资源需要开启静态资源的访问,一般是jQuery、img等，方法如下：  
```xml

<!--    在Spring MVC中开放对哪些资源的访问权限，一般是静态资源-->
    <mvc:resources mapping="/js/**" location="/js/"/>

<!--    或者使用下面的配置,意味着如果Spring MVC
找不到静态资源，交给原始的容器这里是Tomcat找对应的静态资源-->
    <mvc:default-servlet-handler/>
```  
#### 请求数据乱码问题









