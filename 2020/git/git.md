![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/11/1607667705864-1607667705869.png)  
### Git的三种状态  
Git有三种状态，被git管理的文件肯定会处于其中之一的状态。这三种状态分别是：commited(已提交)、modified(已修改)、staged(已暂存)。已提交表示数据安全的存储在本地数据库中。已修改表示文件已经被修改了但是还没有存储到数据库中。已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
由此引入Git项目的三个工作区域的概念：git仓库、工作目录以及暂存区域。如下图所示  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/13/1607842577792-1607842577817.png)  
+ Git仓库目录就是Git用来保存项目的元数据和对象数据库的地方。这时 Git 中最重要的部分，从其他计算既克隆仓库的时候就是拷贝的这里的数据  
+ 工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
+ 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。 有时候也被称作‘索引’，不过一般说法还是叫暂存区域。  
基本Git的工作流程如下  
1. 在工作目录中修改文件  
2. 暂存文件，将文件的快照存入暂存区  
3. 提交更新，找到暂存区域中的文件，将快照永久性的存储到git仓库目录  
如果 Git 目录中保存着的特定版本文件，就属于已提交状态。 如果作了修改并已放入暂存区域，就属于已暂存状态。 如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。  
### Git使用前配置  
Git自带一个`git config`工具来帮助控制设置Git外观和行为的配置变量  
#### 用户信息  
当安装完Git后的第一件事就是应该设置用户名称和邮件地址，这很重要，因为每一个Git的提交都会使用这些信息，并且写入到每一次的提交中不可更改  
```shell
# 注意这里的user.name和user.email是命令的一部分
git config --global user.name username
git config --global user.email user@email.com
```  
如果设置了global，那么只需要设置一次，之后的每次提交都会使用这个信息。当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行不使用 --global 选项的命令来配置
#### 检查配置信息  
如果想要检查你的配置信息可以使用命令`git config --list`命令来列出所有Git当时能找出的配置信息  
### 获取Git仓库  
获取Git仓库有两种办法，第一种是从一个服务器克隆一个现有的Git仓库，另一种是在现有目录或项目下导入所有文件到Git中  
#### 克隆现有的仓库  
如果想要得到一份已经存在Git仓库的拷贝，这时就要用到`git clone`命令。Git克隆的是该Git仓库中的几乎所有数据，而不仅仅是复制完成你工作所需要的数据。当执行`git clone`命令的时候，远程仓库中的每一个文件的每一个版本都会复制到本地。如果远程仓库的磁盘坏掉了，可以使用任何一个克隆下来的用户端重建服务器上的仓库  
例如  
```shell
$ git clone http://git.oschina.net/yiibai/git-start.git
```  
执行这个命令会当前目录下创建一个名为'git-start.git'的目录并在这个目录下初始化一个 .git 文件夹，从远程仓库拉取下所有数据放入 .git 文件夹，然后从中读取最新版本的文件的拷贝。  
如果在克隆的时候想自定义本地仓库的名字，可以使用如下命令  
```shell
$ git clone http://git.oschina.net/yiibai/git-start.git mygit-start
```  
#### 在现有的目录中初始化仓库  
如果不克隆现有的仓库，而是打算使用Git对现有的项目进行管理，只需要进入该目录并执行下面的命令即可  
```shell
$ git init
```  
该命令会创建一个名为.git的子目录，这个子目录含有初始化的Git仓库中的所有的必须文件，这些文件时Git仓库的骨干。但是，在这个时候只是做了一个初始化的操作，项目里的文件还没有被跟踪  
如果是在一个已经存在文件的目录中(不是空文件夹)中初始化Git仓库来进行版本控制的话，应该开始跟踪这些文件并提交。可通过`git add`命令来跟踪文件，通过`git commit`来提交到本地仓库  
现假设目录下有一个文件hello.py，内容如下  
```python
#!/usr/bin/env python
#coding=utf-8

print("This is my first Python Programming")
```  
此时在该目录下执行`git status`命令可以看到如下内容  
```shell
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	hello.py
nothing added to commit but untracked files present (use "git add" to track)
```
这段是提示有一个文件hello.py没有被跟踪，执行`git add`操作用以跟踪hello.py这个文件  
### 更新记录到库  
#### 记录每次更新到仓库  
 工作目录下每一个文件都不外乎这两种状态：已跟踪或未跟踪。已跟踪的文件是指那些被纳入版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态肯处于未修改、已修改或已放入暂存区。工作目录中除了已跟踪文件以外的所有其他文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区中。  
除此克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态  
编辑过某些文件之后，由于自上次提交之后对它做了修改，Git将它们标记为已修改文件。逐步将这些修改过的文件放入暂存区，然后提交所有暂存修改，如此反复。所以使用Git时文件的声明周期如下  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/13/1607870560699-1607870560701.png)  
#### 检查当前文件状态  
要查看文件处于什么状态，可以使用`git status`命令  
如果在克隆后立即使用此命令，可能会看到如下的结果    
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/13/1607871071297-1607871071298.png)  
这说明当前目录非常干净，所有已被跟踪的文件在上次提交之后都没有被修改过  
如果在目录下创建一个新的文件，再执行`git status`命令，将看到一个新的未被跟踪的文件，如下所示  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/13/1607871253744-1607871253751.png)  
这里会看到一个untracked files，表明该文见没有纳入Git的跟踪范围，除非明白地告诉Git需要跟踪该文件  
#### 跟踪新文件  
使用`git add`命令开始跟踪一个新文件  
先使用`git add`命令跟踪一个新文件，然后使用`git status`命令查看这个文件的状态，会发现这个文件已被跟踪，处于暂存的状态  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/13/1607872504316-1607872504317.png)  
上图说明该文件已被跟踪。如果此时提交，那么该文件此时此刻的版本会被保留在历史记录中。`git add`命令使用文件或者目录的路径作为参数，如果参数是目录的路径，该命令会递归地跟踪改目录下的所有文件  
#### 暂存已修改文件  
对已跟踪的文件，进行修改，然后执行`git status`会得到以下结果  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/14/1607910225374-1607910225375.png)  
`Changes not staged for commit`下面的内容说明已跟踪文件的内容发生了变化，但是还没有放入到暂存区，需要执行`git add`命令暂存这次更新。这个命令是多功能命令，可以用它开始跟踪新文件或者把已跟踪的文件放到暂存区，可以将这个命令理解为"添加内容到下一次提交中"  
如果此时向以暂存的文件中添加内容会发生什么？执行`git status`可以看到如下  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/14/1607911129477-1607911129478.png)  
可以发现此时被修改的文件同时出现在了暂存区和工作区。实际上，此时如果执行提交命令，那么存储到本地仓库的内容就是暂存区中的内容，而不是工作区中已修改的内容。如果需要在本地仓库中存储已修改后的内容，需要重新执行`git add`命令，之后再执行`git commit`操作  
#### 忽略文件  
一般总会有些文件无需纳入Git管理，也不希望他们出现在未跟踪列表。这种情况下，可以创建一个名为.gitignore的文件，列出要忽略的文件模式。要养成以开始就设置好.gitignore文件的习惯，以免将来误提交这类无用的文件  
#### 查看已暂存和未暂存文件的修改  
要查看尚未暂存的文件更新了哪些部分，不加参数直接输入`git diff`  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/14/1607912997692-1607912997693.png)  
上面红色的部分前面带有一个"+"号，表示该文件添加了一行  
请注意，`git diff`本身只是显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。
如果要查看已暂存起来的变化，可以使用`git diff --cached`或者`git diff --staged`命令  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/14/1607913421222-1607913421223.png)  
在上图中，"+"表示添加了一行，"-"表示删除了一行  
#### 提交更新  
在提交更新之前，一定要确认还有什么修改过的或新建的文件没有`git add`过，否则提交的时候不会记录这些还没有暂存起来的变化。所以，在每次提交之前，先用`git status`看一下文件是不是都已暂存起来的了，如果没有暂存起来则先要使用命令`git add .`将所有的文件暂存起来，然后再运行`git commit`提交  
#### 跳过使用暂存区  
尽管使用暂存区的方式可以精心准备要提交的细节，但有时候这么做还是略嫌麻烦。Git提供了一种跳过使用暂存区的方式，即`git commit -a`命令，使用这个命令会自带把所有已经跟踪过的文件暂存起来一并提交，从而跳过`git add`的步骤  
#### 移除文件  
要从Git中移除某个文件，就必须要从已跟踪文件清单中移除(确切地说，是从暂存区移除)，然后提交。可以用`git rm`命令完成此项工作，并连带从工作目录中删除指定文件，这样以后就不会出现在未跟踪文件清单中了  
如果删除之前文件已经被修改过并且已经存放到暂存区的话，就必须要用强制删除选项-f，这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被Git恢复  
另外一种情况是，想把文件从Git仓库中删除(亦即从暂存区中移除)，但仍然希望保留在当前工作目录中。换句话说，就是继续保留文件在磁盘中，但是不像让Git继续追踪。为达到这一目的，可以使用--cached选项  
#### 移动文件  
要在Git中对文件改名，可以这么做  
`git mv file_from file_to`  
它会恰如预期般正常工作，实际上，即便此时产看状态，也会明白无误地看到关于重命名地操作  
```shell
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
    renamed:    README.md -> README
```  
其实，运行`git mv`就相当于运行了下面三条命令  
```shell
$ mv README.md README
$ git rm README.md
$ git add README
```  
即便如此操作，Git也会意识到这是一次重命名操作  
### 查看历史提交  
运行`git log`命令，会按照时间列出所有地更新，最近的更新排列在最上面
一个常用的选项是-p，用来显示每次提交的内容差异；--stat选项在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或者是添加了；--pretty=oneline，将每个提示放在一行显示  
### 撤销操作  
需要注意的是，有些撤销操作是不可逆的，这是在使用Git的过程中，会因为操作失误而导致之前的工作丢失的少有的几个地方之一  
#### 重新提交  
有时候提交完了发现漏掉了几个文件没有添加，或者提交信息写错了。此时，可以运行带有`--amend`选项的提交命令尝试重新提交  
`git commit --amend`  
这个命令会将暂存区的文件提交，如果自上次提交以来还未做任何修改(例如，在上次提交之后马上执行了此命令)，那么快照会保持不变，而所修改的只是提交信息  
例如，提交后发现忘记了暂存某些需要的修改，可以像下面这样操作  
```shell
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```  
最终只有一个提交，第二次提交将替代第一次提交的结果  
#### 取消暂存的文件  
如果有两个文件本来想要作为两次独立的修改提交，但是却意外的输入了`git add *`暂存了他们两个。那么如何取消只暂存一个呢？可以输入`git status`命令提示  
```shell
$ git add *
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    README.md -> README
        deleted:    mytext.txt
```  
从`Changes to be committed:`下面的提示可以看到，`git reset HEAD <file>...`命令来取消暂存  
#### 撤销对文件的修改  
如何不想保存对文件的修改该怎么办？幸运的是，`git status`命令明确的告诉了我们该如何做  
```shell
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    README.md -> README

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    mytext.txt
```  
`Changes not staged for commit`后面提示我们可以使用`git checkout -- <file>`来取消对工作区中文件的修改  
在Git中，任何已提交的东西几乎总是可以恢复的。然而，未提交的东西有可能会在丢失后再也找不回来了  
### 远程仓库的使用  
#### 查看远程仓库  
如果想查看已经配置的远程仓库，可以运行`git remote`命令，该命令会列出指定的每一个远程服务器的简写  
也可以指定选项`-v`,会显示需要读写远程仓库使用的Git保存的简写与其对应的URL  
```shell
$ git remote -v
mydoor  http://git.oschina.net/yiibai/git-start.git (fetch)
mydoor  http://git.oschina.net/yiibai/git-start.git (push)
curry     http://git.oschina.net/yiibai/git-start.git (fetch)
curry     http://git.oschina.net/yiibai/git-start.git (push)
deepfun   http://git.oschina.net/yiibai/git-start.git (fetch)
deepfun   http://git.oschina.net/yiibai/git-start.git (push)
koke      http://git.oschina.net/yiibai/git-start.git (fetch)
koke      http://git.oschina.net/yiibai/git-start.git (push)
```  
如果远程仓库不止一个，该命令会将它们全部列出  
#### 添加远程仓库  
使用命令`git remote add <shortname> <URL>`添加一个远程的Git仓库，同时指定一个URL的简写  
```shell
$ git remote
origin

$ git remote add gs http://git.oschina.net/yiibai/git-start.git
$ git remote -v
gs      http://git.oschina.net/yiibai/git-start.git (fetch)
gs      http://git.oschina.net/yiibai/git-start.git (push)
origin  http://git.oschina.net/yiibai/git-start.git (fetch)
origin  http://git.oschina.net/yiibai/git-start.git (push) 
```  
可以在命令行中使用简写来替代整个URL  
#### 从远程仓库抓取与拉取  
`git fetch [remote-name]`  
这个命令会访问远程仓库，从中拉取所有还没有的数据。执行完后，将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看  
注意  
**`git fetch` 命令会将数据拉取到本地仓库，它并不会自动合并或修改当前的工作。当准备好时必须手动将其合并入自己的工作区**  
如果有一个分支设置为跟踪一个远程分支，可以使用`git pull`命令来自动抓取然后合并远程分支到当前分支  
默认情况下，`git clone`命令会自动设置本地master分支跟踪克隆远程仓库的master分支。运行`git pull`通常会从最初克隆的服务器上抓取数据自动尝试合并到当前所在的分支  
#### git fetch和git pull的区别  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608003042789-1608003042790.png)  
根据上图，`git fetch`和`git pull`都会将远程分支上的commit拉取到本地仓库；简单来讲，`git fetch`相当于`git pull`一半的操作，另一半则是合并  
#### 推送到远程仓库  
使用命令`git push [remote-name] [branch-name]`将本地项目推送到远程仓库  
`git push origin master`是将本地master分支推送到origin服务器。只有当你有服务器的写权限并且之前没有人推送过时，这条命令才能生效。当和其他人一起克隆，别人先推送你再推送时，该推送毫无疑问会被拒绝。必须先将别人的项目拉下来并合并进你的项目后才能再推送  
`git push <remote-name> <local_branch_name>:<remote_branch_name>`
#### 查看远程仓库  
如果想要查看某一个远程仓库的更多信息时，可以使用`git remote show [remote-name]`命令  
#### 远程仓库的移除与重命名  
如果想要重命名引用的名字，可以运行`git remote renmae old_name new_name`  
值得注意的是这同样也会修改远程分支的名字。过去那些引用old_name/master的现在会引用new_name/master  
如果想要移除一个远程仓库，可以使用`git remote rm`  
### Git分支  
首先得知道，Git分支包括本地分支和远程分支  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608018649261-1608018649263.png)  
以下具体说明，如何创建本地分支和远程分支  
#### 创建本地分支  
新分支都是基于原有分支创建，而在实践开发中基本都是从master分支创建  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608018758034-1608018758035.png)  
从master分支创建本地分支也有两种方式  
##### 基于本地master分支创建分支  
查看当前分支是否在master分支  
`git branch`  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608019145224-1608019145226.png)  
绿色分支表示其为当前分支，所以得切换至master分支  
```shell
#切换至master分支
git checkout master
#更新本地master代码至最新，如果本地master分支没有关联远程master分支，git pull origin master
git pull 
#基于本地master分支创建新分支，并切换至创建的新分支  
git checkout -b newBranchName
```  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608019323168-1608019323169.png)
##### 基于远程master分支创建分支  
首先查看本地、线上分支信息(调用以下命令前，建议先执行"git pull -p"防止本地git分支缓存)    
`git branch -a`  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608020403515-1608020403516.png)  
其中，白色显示为本地分支、绿色显示为当前分支、红色显示为远程分支  
```shell
# 切换至远程分支  
git checkout remotes/origin/master
# 基于远程master分支创建新分支
git checkout -b newBranchName 
```  
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608020569803-1608020569804.png)
#### 创建远程分支  
创建远程分支可以直接由本地分支推送完成也可以在远程分支管理系统(github、gitlab)上可视化操作完成  
##### 本地新分支推送创建远程分支  
```shell
# git push <远程主机名> <本地分支名>:<远程分支名>
git push origin newBranchName:newBranchName
```
![title](https://raw.githubusercontent.com/liujinxi931204/image/master/gitnote/2020/12/15/1608020929174-1608020929175.png)
##### 本地分支关联远程分支  
###### 如果本地创建了一个分支而远程没有该分支  
```shell
# git branch --set-upstream-to=origin/<远程分支名> <本地分支名>
# 如设置当前分支，第二个参数可以省略不写
git branch --set-upstream-to=origin/<branch> newBranchName
# --set-upstream-to等价于-u，所以上述可以写成
git branch -u=origin/<branch> newBranchName
```  
注：关联本地分支和远程分支，可以直接使用git pull或者git push命令，而不需要指定远程分支  
###### 如果远程新建了一个分支而本地没有该分支  
```shell
git checkout --track origin/branch_name
```  
这时本地会创建一个名叫branch_name的分支，自动跟踪远程的同名分支  
### Git切换分支  
#### 工作区没新代码切换分支 
创建好分支以后就可以在新分支进行开发了，但可能需要中途去维护其他分支代码；这个时候就得切换分支了，切换分支指令  
```shell
git checkout newBranchName 
```  
编辑代码不会直接在develop、master分支操作，因为最终代码要同时合并到这两个分支上，所以一般均在新分支上开发(即使是很小的改动)  
#### 工作区有新代码切换分支  
**如果在工作区中修改了代码，但是没有执行add或者commit或者stash，这时切换分支会报错；如果在工作区修改了代码并且执行了add，但是没有执行commit，这时切换分支，会将修改的内容一起带过去**  
如果工作区中有新修改的代码，切换分支时一般会这样  
```shell
# 如果直接git stash 则将上次commit注释作为说明
# 使用‘’
git stash save "存储说明"

```













