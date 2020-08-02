# Git  
## 配置git仓库
配置git仓库有两种方式    
1. 将已有的项目纳入git管理  
```shell
cd your_project  
git init  
```  
2. 新建项目直接直用git管理  
```shell
cd some_dic
git init your_project
cd your_project
```  
## 配置用户名和email标记信息  
1.默认配置是local信息，local值只对当前仓库有效  
```shell
git config --local user.name "username"
git config --local user.email "user@email.com"
```
2.配置global信息，global信息对当前登陆用户有效  
```shell
git config --global user.name "username"  
git config --global user.email "user@email.com"  
```  
3.配置system信息，system信息对系统所有登陆用户有效，一般基本不用  
```shell
git config --system user.name "username"  
git config --system user.email "user@email.com"  
```




