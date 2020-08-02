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
## 配置user信息  
1.默认配置是local信息，local值只对当前仓库有效  
```shell
git config --local user.name "username"
git config --local user.email "user@email.com"
```





