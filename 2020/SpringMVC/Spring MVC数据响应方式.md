### Spring MVC的数据响应方式  
1. 页面跳转  
+ 直接返回字符串  
+ 通过ModelAndView对象返回  
  
2. 回写数据  
+ 直接返回字符串  
+ 返回对象或集合  
  
### 页面跳转  
1. 直接返回字符串  
直接返回字符串：此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转  
```java
@ResultMapping("/quick")
public String quickMethod(){
    
    return "index"
}
```  
```xml

```