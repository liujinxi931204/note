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
Controller中的业务方法