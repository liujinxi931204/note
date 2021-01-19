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
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/24/1590311881370-1590311881374.png)
### cookie的创建的代码实现
```java

public void doGet(HttpServletRequest req, HttpServletResponse resp){
    //向客户端写cookie
    Cookie cookie=new Cookie(String,String);
    //创建cookie，key:value是必填的
    cookie.setMaxAge(int);
    /用来设置cookie的有效期，时间单位是秒。如果是负数意味着，永远不删除cookie，如果是正数，表示有效期到正数秒，如果为0，表示立即失效
    cookie.setPath(String);
    //默认情况可以不设置，以为着url路径下的所有cookie都可以被访问
    resp.addCookie(cookie);
    //最后这一句必须要写，否则cookie不会创建
}

public void doGet(HttpServletRequest,HttpServletResponse resp){
    //服务器得到传来的cookie
    Cookie[] cookies = req.getCookies();
    //这个方法会得到所有的cookie，没有单独获得一个cookie的方法
    for (Cookie cookie1 : cookies) {
            System.out.println(cookie1.getName());
            System.out.println(cookie1.getValue());
        }

} 
```
### session的创建过程  
1.session是基于cookie的，所以首先要产生cookie。在client访问server的时候，server会随机产生一个sessionId，并将其放在响应头中，以cookie的形式返回给client    
2.server会将需要保存的数据存放在对应sessionId的之下，将sessionId保存在内存中  
3.client再次访问server的时候会带上sessionId在cookie中，server获取到sessionId,然后在内存中查找，如果找到就返回信息给client  
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/24/1590311982792-1590311982797.png)
### session的创建的代码实现  
```java
public void doGet(HttpServletRequest req,HttpServletReponse resp){
    HttpSession session = req.getSession();
    //使用这个方法，如果session不存在，则会自动创建一个session
    session.setAttribute("key","value");
        //将信息存储到session中
        if(session.isNew()){
            //在第一次创建的时候会返回true，之后会返回false，除非重新创建
            System.out.println("新的");

        }else{
            System.out.println("旧的");
        }
        session.setMaxInactiveInterval(30*60);
        //设置session的过期时间为30*60秒
        //如果为负值，表示永远不销毁
        session.invalidate();
        //立即销毁session

}
```
![title](https://gitee.com/liujinxi931204/image/raw/master/gitnote/2020/05/24/1590311942442-1590311942445.png)  


### session和cookie的区别  
1.session是保存在server端的，而cookie是保存在client端的  
2.session用户无法查看和修改，cookie用户则可以查看和修改  
3.session和cookie的存储容量不同，单个cookie保存数据的容量不能超过4KB
4.session的实现依赖于sessonId,sessionId又是存储在cookie中的，可以说session是基于cookie的  


