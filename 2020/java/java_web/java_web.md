# java_web
## cookieh和session 
cookie和session就是为了解决HTTP协议是无状态的协议  
### cookie的创建过程   
client指客户端，server指服务端  
1.第一次client第一次访问server的时候，server会在响应头中设置一个cookie返回给client，其中保存cookie的内容。即set-cookie  
2.client在接受到server返回的cookie之后，会保存这个cookie并设置一个有效期，过了有效期这个cookie会失效  
3.client再次访问server的时候会在请求头中带上保存的cookie，将cookie传递到server  
4.server接收到cookie之后，会解析其中的内容，并将相应的信息返回给client  
在cookie没有失效之前都是围绕着2-4步来进行的  
