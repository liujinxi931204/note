## SpringMVC概述  
SpringMVC是一种基于java实现的MVC设计模式的请求驱动类型的轻量级的WEB框架，即使用MVC架构模式的思想，将web层进行职责解耦，基于驱动请求指的就是请求--响应模型，属于Spring FrameWork的后续产品，已经融合到Spring Web Flow中  
  
### Spring MVC处理请求的流程  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/10/22/1603337222739-1603337222741.png)  
  
具体执行步骤如下：  
1. 首先用户发送请求---->前端控制器，前端控制器根据请求信息(如URL)来决定选择哪一个页面控制器进行处理并把请求委托给它，即一千的控制器的控制逻辑部分；图中的1、2步骤  
  
2. 页面控制前接收到请求后，进行功能处理，首先需要收集和绑定请求参数到一个对象，这个对象在Spirng Web MVC中叫命令对象，并进行验证，然后将命令对象委托给业务对象进行处理；处理完成后返回一个ModelAndView(模型数据和逻辑视图名)；图中的3、4、5步骤  
  
3.  前端控制器收回控制权，然后根据返回的逻辑视图名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；图中的6，7步骤  
  
4. 前端控制器再次收回控制权，将响应返回给用户，图中的步骤8  
  
### Spring MVC核心架构  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/10/22/1603337268750-1603337268751.png)  
  
核心架构的具体流程步骤如下：  
1. 首先用户发送请求---->DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器处理，作为统一的访问点，进行全局的流程控制  
  
2. DispatcherServlet---->HandlerMapping,HandlerMapping将会把请求映射为HandlerExecutionChain对象(包含一个Handler处理器(页面控制器)对象、多个HandlerInterceptor拦截器)对象，通过这种策略模式，很容易添加新的映射策略  
  
3. DispatcherServlet---->HandlerAdapter，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器  
  
4. HandlerAdapter---->处理器功能处理方法的调用，HandlerAdapter将会根据
  
Spring MVC目前已经称为最主流的的MVC的框架之一。它通过一套注解，让一个简单的Java类成为处理请求的控制器，而无需实现任何接口。同时它还支持RESTful编程风格的请求  
### Spring MVC开发步骤  
1. 导入Spring MVC相关坐标  
2. 配置Spring MVC核心控制器(前端控制器)DispathcerServlet  
3. 创建Crontroller类和视图  
4. 使用注解配置Controller类中业务方法的映射地址  
5. 配置Spring MVC核心配置文件spring-mvc.xml  
6. 客户端发起请求测试  
