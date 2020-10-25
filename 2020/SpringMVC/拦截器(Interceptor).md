### 拦截器(Interceptor)的作用  
Spring MVC的拦截器类似于servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理  
  
将拦截器按照一定的顺序连结成一条链，这条链成为连接器链(Interceptor Chain)。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按照其之前定义的顺序被调用。拦截器也是AOP思想的