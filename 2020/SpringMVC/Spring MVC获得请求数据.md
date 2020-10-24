### 获得请求参数  
客户端请求参数的格式为：name=value&name=value&...  
服务器端要获得请求的参数，有时还需要进行数据的封装，Spring MVC可以接受如下类型的参数  
+ 基本类型参数  
+ POJO类型参数(简单的java bean)  
+ 数组类型参数  
+ 集合类型参数  
### 获得基本类型参数  
Controller