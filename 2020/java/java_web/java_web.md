# java_web
## cookieh和session 
cookie和session就是为了解决HTTP协议是无状态的协议  
### cookie的创建过程   
client指客户端，server指服务端  
1.第一次client第一次访问server的时候，server会在响应头中设置一个cookie返回给client，其中保存cookie的内容。即set-cookie，