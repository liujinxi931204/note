![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/11/1607667705864-1607667705869.png)  
# Git  
## 配置git仓库
配置git仓库有三种方式    
1. 将已有的项目纳入git管理  
```shell
cd your_project  
git init  
```  
2. 新建项目直接直用git管理  
```shell
cd some_dir
git init your_project
cd your_project
```  
3. 远程clone  
```shell
cd some_dir  
git clone [url]  
```
## 配置用户名和email标记信息  
1. 默认配置是local信息，local值只对当前仓库有效  
```shell
git config --local user.name "username"
git config --local user.email "user@email.com"
```
2. 配置global信息，global信息对当前登陆用户有效  
```shell
git config --global user.name "username"  
git config --global user.email "user@email.com"  
```  
3. 配置system信息，system信息对系统所有登陆用户有效，一般基本不用    
```shell
git config --system user.name "username"  
git config --system user.email "user@email.com"  
```  
## 查看当前配置信息  
1. 查看local配置信息  
`git config --local --list`  
2. 查看global配置信息  
`git config --global --list`  
3. 查看system配置信息  
`git config --system --list`  
## git管理  
### 首次纳入git管理或者工作区提交到暂存区  
`git add filename`  
### 暂存区提交到本地库  
`git commit -m "提交的理由"`
### 工作区直接提交到本地库，不经过暂存区，不提倡  
`git commit -am "提交的理由"`  
### 查看状态  
`git status`  
### 更改文件名  
`git mv old_file_name new_file_name` 
### 查看历史  
#### 查看当前分支所有提交的历史  
`git log`  
#### 查看当前分支最近number行提交的历史  
`git log -nnumber`   
#### 在一行内展示所有当前分支的提交历史  
`git log --oneline`  
#### 在一行内以图形的方式展示当前分支的所有提交历史  
`git log --oneline --graph`  
#### 在一行内以图形的方式展示所有分支的提交历史  
`git log --online --graph --all`  
### 分支  
#### 创建分支  
1. 创建分支但没有切换过去  
`git branch '分支名'`  
2. 创建分支并切换过去  
`git checkout -b '分支名'`  
#### 切换分支  
`git checkout '分支名'`  











