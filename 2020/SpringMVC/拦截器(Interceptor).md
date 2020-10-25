### 拦截器(Interceptor)的作用  
Spring MVC的拦截器类似于servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理  
  
将拦截器按照一定的顺序连结成一条链，这条链成为连接器链(Interceptor Chain)。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按照其之前定义的顺序被调用。拦截器也是AOP思想的体现  
  
#### 拦截器和过滤器的区别  
|区别|过滤器|拦截器|
|-|-|-|
|使用范围|是servlet规范中的一部分，任何java web工厂都可以使用|是spring mvc框架自己的，只有使用了Spring MVC框架的工厂才能使用|
|拦截范围|在url-pattern中配置了/*之后，可以对所有要访问的资源拦截|只会拦截访问的控制器方法，如果访问的是jsp，html，css，image或者js是不会进行拦截的|  
#### 拦截器的快速入门  
自定义拦截器只有三步  
1. 创建拦截器类实现HandlerIntercetor接口  
2. 配置拦截器  
3. 测试拦截器效果  
+ **创建拦截器类**  
```java
package com.sogou.Intercepor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * author liujinxi@sogou-inc.com
 * date 2020-10-25 21:56
 **/
public class myInterceptor implements HandlerInterceptor {
//    在目标方法之前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("perHandler...");
        return false;
    }

//    在目标方法执行之后，视图对象返回之前执行
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

        System.out.println("postHandler...");
    }

//    在流程都执行完毕后执行
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

        System.out.println("afterHandler...");
    }
}

```  
+ **配置拦截器**  
在sping-mvc.xml中配置  
```xml
<!--    配置拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
<!--
            <mvc:mapping path="/**"/>
            <bean class="com.sogou.Intercepor.myInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```
