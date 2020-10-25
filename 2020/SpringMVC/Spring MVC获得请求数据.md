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
当post请求时，数据会出现乱码，可以设置一个过滤器来进行编码的过滤  
在web.xml中进行如下配置  
```xml
<!--  配置全局过滤器,解决POST乱码问题-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```
#### 参数绑定注解@requestParam  
当请求的参数名称与Controller的业务方法参数名称不一致时，就可以通过@requestParam注解显式的绑定  
```jsp
<form action="${pageContext.request.contextPath}/quick" method="post">
    <input type="txt" name="name"/><br>
    <input type="sumbit" value="提交"/><br>
</form>
```  
```java
@RequestMapping("/quick")
@ReponseBody
public void quickMethod(@requestParam("name")String username) thorws IOExecption {
    System.out.println(username);
}
```  
此时可以在浏览器中输入下地址：http://localhost:8080/quick?name="xxx"，通过上述的@requestParam注解，可以将name的参数绑定到username中  
  
注解@requestParam注解还有如下参数  
+ **value**:请求参数名称  
+ **required**：此在指定请求参数是否必须包含，默认为true，意味着如果没有该参数，会报错  
+ **defaultValue**：当没有指定请求参数时，则使用指定的默认值进行赋值  
#### 获得Restful风格的参数  
Restful是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件按。主要用于客户端和服务端交互类的软件，基于这种风格的软件可以更简洁，更有层次，更易于实现缓存机制等  
Restful风格的i请求是使用"URL+请求方式"表示一次请求的目的，HTTP协议里四个表示方式的动词如下：  
+ GET: 用于获取资源  
+ POST：用于新建资源  
+ PUT：用于更新资源  
+ DELETE：用于删除资源  
例如：  
+ /user/1 GET：得到id=1的user  
+ /user/1 DELETE：删除id=1的user  
+ /user/1 PUT：更新id=1的user  
+ /user POST：新增user  
上述url地址/user/1中的1就是要获得的请求参数，在Spring MVC中可以使用占位符进行参数绑定。地址/user/1可以写成/user/{id},占位符{id}对应的1就是指。在业务方法中可以使用@PathVariable注解进行占位符的匹配获取工作  
`http://localhost:8080/quick/1`  
```java
@RequestMapping("/quick/${id}")
@ReponstBody
public void quickMethod(@PathVariable(value="id",required=true)int id){
    System.out.println(id);
}
```  
#### 自定义类型转换器  
+ Spring MVC默认已经提供了一些常用的类型转换器，例如客户端提交的字符串转换成int类型进行参数设置  
+ 但是不是所有的数据类型都提供了转换器，没有提供的就需要自定义转换器，例如：日期类型的数据就需要自定义类型转换器  
  
自定义类型转换器的步骤  
+ 自定义转换器类实现Converter接口  
+ 在配置文件spring-mvc.xml中声明转换器  
+ 在<annotion-driven>中引用转换器  
  
1. 自定义转换器类实现Converter接口  
```java
public class dateConverter implements Converter<String,Date> {
    @Override
    public Date convert(String str) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = simpleDateFormat.parse(str);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
```
2. 在配置spring-mvc.xml中声明转换器  
```xml
<!--    声明自定义类型转换器-->
    <bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.sogou.service.dateConverter"></bean>
            </list>
        </property>
    </bean>
```  
3. 在<annotion-driven>中引用转换器  
```xml
<mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
```
#### 获得请求头  
使用@RequestHeader可以获得请求头信息  
@RequestHeader注解的属性如下：  
+ value：请求头的名称  
+ required：是否必须携带此请求头  
```java
@RequestMapping("/quick")
@ReponseBody
public void quickMethod(@RequestHeader(value="User-Agent",required=false) String userAgent) throws IOException{
    System.sout.println(userAgent);
}
```  
**@CookieValue**  
使用@CookieValue可以获得指定Cookie的值  
@CookieValue注解的属性如下：  
+ value：指定cookie的名称  
+ required：是否必须携带此cookie  
```java
@RequestMapping("/quick")
@ReponseBody
public void quickMethod(@CookieValue(value="JSESSIONID",required=false)String jsessionid){
    System.out.println(jsessionid);
}
```  
#### 文件上传  
##### 文件上传客户端三要素  
+ **表单项type="file"**  
+ **表单的提交方式是post**  
+ **表单的enctype属性是多部份表单形式，以及enctype="multipart/form-data"**  
```jsp
<form action="${pageContext.request.contextPath}/quick" method="post" enctype="multipart/form-data">
    名称:<input type="text" name="name"><br>
    文件:<input type="file" name="file"><br>
    <input type="sumbit" value="提交"><br>
</form>
```  
##### 文件上传原理  
+ 当form表单项修改为多部份表单时，request.getParameter()方法将失效  
+ enctype="application/x-www-form-urlencode"时，form表单的正文内容格式是key=value&key=value...的形式  
+ enctype="mutlipart/form-data"时，请求正文内容就变成多部份形式：  
```shell
   Content-Type: multipart/form-data; boundary=AaB03x
   --AaB03x
   Content-Disposition: form-data; name="submit-name"
   Larry
   --AaB03x
   Content-Disposition: form-data; name="files"; filename="file1.txt"
   Content-Type: text/plain
   ... contents of file1.txt ...
   --AaB03x--
```
##### 单文件上传步骤  
+ 导入fileupload和io坐标  
+ 配置文件上传解析器  
+ 编写文件上传代码  
  
1. 导入fileupload和io的坐标 
```xml
<dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.3</version>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.4</version>
    </dependency>
```
2. 配置文件上传解析器  
```xml
<bean id="commonsMultipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--       上传文件总大小-->
        <property name="maxUploadSize" value="5242800"/>
<!--        上传单个文件的大小-->
        <property name="maxUploadSizePerFile" value="5242800"/>
<!--        上传文件的编码类型-->
        <property name="defaultEncoding" value="UTF-8"/>
    </bean>
```
3. 编写文件上传代码  
```java
@RequestMapping("/quick")
    @ResponseBody
    public void quickMethod(String name, MultipartFile file) throws IOException {
       //这里的upload对应的是表单中的file的name，所以这里是"file"
        String originalFilename = file.getOriginalFilename();
      //保存文件
        file.transferTo(new File("C:\\upload\\"+originalFilename));

    }
```  
#### 多文件上传  
多文件上传将单文件上传表单项、上传文件代码多写一份即可  
```jsp
<form action="${pageContext.request.contextPath}/quick" method="post" enctype="multipart/form-data">
    名称:<input type="text" name="name"><br>
    文件:<input type="file" name="file"><br>
    文件:<input type="file1" name="file"><br>
    <input type="sumbit" value="提交"><br>
</form>
```    
```java
@RequestMapping("/quick")
    @ResponseBody
    public void quickMethod(String name, MultipartFile file, MultipartFile file1) throws IOException {
       //这里的upload对应的是表单中的file的name，所以这里是"file"
        String originalFilename = file.getOriginalFilename();
      //保存文件
        file.transferTo(new File("C:\\upload\\"+originalFilename));
        String originalFilename1 = file1.getOriginalFilename();
        file1.transferTo(new File("C:\\upload\\"+originalFilename1));

    }
```




