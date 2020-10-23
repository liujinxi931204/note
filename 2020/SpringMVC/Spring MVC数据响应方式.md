### Spring MVC的数据响应方式  
1. 页面跳转  
+ 直接返回字符串  
+ 通过ModelAndView对象返回  
  
2. 回写数据  
+ 直接返回字符串  
+ 返回对象或集合  
  
### 页面跳转  
####  直接返回字符串  
直接返回字符串：此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转  
```java
@RequestMapping("/quick")
public String quickMethod(){
    
    return "index"
}
```  
在Spring MVC的配置文件spring-mvc.xml中配置  
```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
</bean>
```  
根据以上的配置，最终的转发地址为/WEB-INF/views/index.jsp  
方法quickMethod的默认结果是转发，也可以写成  
```java
@RequestMapping("/quick")
public String quickMethod(){
    
    return "forword:/WEB-INF/views/index.jsp"
}
```  
这种返回方式和默认返回字符串一样都是转发  
```java
@RequestMapping("/quick")
public String quickMethod(){
    
    return "redirect:/index.jsp"
}
```  
**这里redirect和forwad的URL不一样，是因为redirect是重新访问服务器，默认WEB-INF下的内容是受保护的内容不可以被访问，因此使用redirect时必须保证后面的地址是有权限被访问的**    
  
#### 返回ModelAndView  
```java
@RequestMapping("/quick")  
public ModelAndView quickMethod(){
    /*
        Model 模型  封装数据  
        View  视图  展示数据
    */
    ModelAndView modelAndView = new ModelAndView();
    //ModelAndView设置模型数据,相当于数据放入到request域中  
    modelAdnView.addObject("username","xxx")
    //ModelAndView设置视图数据
    modelAndView.addViewName("index");
    return modelAndView;
}
```  
### 回写数据  
#### 直接返回字符串  
+ 通过Spring MVC框架注入response对象，使用response.getWriter().print("hello world")回写数据，此时不需要视图跳转，业务方法返回值为void    
```java
@RequestMapping("/qiuck")
public void quickMethod(HttpServletResponse response) throw IOException{
   response.getWriter().print("hello world");
}
```  
+ 将需要的字符串直接返回，但此时需要通过@ResponseBody注解告知Spring MVC框架，方法返回的字符串不是跳转而是直接在http响应体中返回  
```java
@RequestMapping("/quick")
@ResponseBody//该方法不进行视图跳转，直接在http响应中返回
public String quickMethod(){
    return "hello world";
}
```  
#### 返回对象和集合  
##### 返回json格式  
1. pom.xml中引入相关依赖  
```xml
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.11.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.11.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.11.0</version>
    </dependency>
```  
2. spring-mvc.xml配置文件中配置处理器适配器  
```xml
    <bean id="requestMappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean id="mappingJackson2HttpMessageConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"></bean>
            </list>
        </property>
    </bean>
```  
3. 编写controller  
```java
@RequsetMapping("/quick")
@ReponseBody
public User quickMethod(){
    User user = new User();
    user.setId(1);
    user.setName("xxx");
    return user;
}
```  
这样就会在浏览器页面返回一个json格式的字符串  
***  
上述返回对象或集合的方式配置较为繁琐，可以使用mvc的注解驱动代替上述配置  
在Spring MVC的各个组件中，处理器适配器、处理器映射器、视图解析器成为三大组件。使用<mvc:annotion-driven>自动加载RequestMappingHandlerMapping（处理器映射器）和RequestMappingHandlerMapping（处理器适配器），可用在spring-mvc.xml配置文件中使用代替处理器适配器和处理器

