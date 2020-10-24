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
+ 获得集合参数是，要将集合参数包装到一个POJO中才可以  
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










