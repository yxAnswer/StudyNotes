# Git常用命令总结

git有这么几个概念：

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

常用命令，先掌握这些再说：

## 1、初始化、配置

```shell
$ git init				# 在当前目录新建一个Git代码库
$ git init [project-name]# 新建一个目录，将其初始化为Git代码库
$ git config --list	# 显示当前的Git配置
$ git config [--global] user.name "[name]"  # 设置提交代码时的用户信息
$ git config [--global] user.email "[email address]"
```



## 2、添加、提交、删除

```shell
$ git add [file1] [file2] ...   # 添加指定文件到暂存区
$ git add [dir]   # 添加指定目录到暂存区，包括子目录
$ git add .  # 添加当前目录的所有文件到暂存区
$ git commit -m [message]  # 提交暂存区到仓库区
$ git commit [file1] [file2] ... -m [message]  # 提交暂存区的指定文件到仓库区
$ git rm [file1] [file2] ...	# 删除工作区文件，并且将这次删除放入暂存区
$ git rm -r [文件夹]  #删除文件夹下所有
$ git rm --cached [file] 	# 停止追踪指定文件，但该文件会保留在工作区
$ git mv a.txt b.txt	# 重命名文件并直接提交到暂存区
```



## 3、查看信息

```shell
$ git status	# 查看状态
$ git log		# 显示当前分支的版本历史
$ git log --all		# 显示所有日志
$ git log --praph		# 有些图形化的方式现实
$ git log --oneline #单行显示所有提交历史
$ git log --pretty=oneline  # --pretty可以做一些格式化，接一些其他属性
$ git log --pretty=oneline --abbrev-commit 
$ git log -5  --oneline 	# 单行显示过去5次所有提交
$ git shortlog -sn	# 显示所有提交过的用户，按提交次数排序
$ git blame [file]	# 显示指定文件是什么人在什么时间修改过
$ git diff			# 显示工作区和暂存区的区别
$ git diff HEAD		# 显示工作区与当前分支最新commit之间的差异（工作区和本地仓库区别）
$ git diff HEAD -- [file]# 显示工作区某一个文件与当前分支最新commit之间的差异（工作区和本地仓库区别）
$ git diff --cached [file]# 显示暂存区和上一个commit的差异（暂存区和本地仓库区别）
$ git diff [first-branch]...[second-branch] # 显示两次提交之间的差异
$ git show [commit版本号]	# 显示某次提交的元数据和内容变化
$ git reflog	# 显示当前分支的最近几次提交
```



## 4、分支

```shell
$ git branch	# 列出所有本地分支
$ git branch -r # 列出所有远程分支
$ git branch -a	# 列出所有本地分支和远程分支
$ git branch -v# 查看分支的最后一次提交
$ git branch -av #列出所有分支并查看最后一次提交

$ git branch [branch-name]	# 新建一个分支，但依然停留在当前分支
$ git checkout -b [branch]	# 新建一个分支，并切换到该分支
$ git switch -c [branch]	# 新建一个分支，并切换到该分支--新版本有
$ git checkout [branch-name] # 切换到指定分支
$ git switch [branch-name] # 切换到指定分支--新版本

$ git merge [branch]	# 合并指定分支到当前分支
$ git merge --no-ff  -m '日志' [branch]  //使用 --no-ff 方式合并,在删除分支后会保留提交信息
$ git cherry-pick [commit]	# 选择一个commit，合并进当前分支
$ git branch -d [branch-name]	# 删除分支（本地的）

# 删除远程分支--两种
$ git push origin --delete [branch-name]
$ git push origin :master
#查看日志分支提交记录，分支合并图
$ git log --graph --pretty=oneline   
```



## 5、版本回退（撤销）

```shell
# 暂存区————>工作区
$ git checkout -- [file]	# 恢复暂存区的指定文件到工作区
$ git checkout .	# 恢复暂存区的所有文件到工作区
# 版本库————>暂存区
$ git reset head [file] #重置暂存区，与当前版本库一致
# 版本库————>重置暂存区 和工作区
$ git reset --hard^	# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard [commit]# 回滚到指定commit，重置暂存区和工作区
git
# 当在本分支没写完不能提交时，要切换分支，可以使用stash  先保存
$ git stash    #保存工作区，然后去其他分支改bug
$ git stash list   #查看存储记录
$ git stash apply  [可以是指定的]   #恢复工作区
$ git stash pop  [可以是指定的] #恢复工作区,并删除stash记录
$ git cherry-pink [commit]	 #版本号-复制其他分支一次提交到本分支。只会复制这一次提交修改的内容
```



## 6、远程仓库、多人协作

```shell
$ git clone    		 # 克隆项目到本地
$ git fetch [remote] # 下载远程仓库的所有变动
$ git remote   		 #查看远程库信息
$ git remote  -v 	 #查看详细远程仓库信息 
$ git push origin master   #推送本地master到远程分支master
$ git push origin dev      #推送本地dev  到远程dev分支（名字自己取）

$ git pull    #下拉远程分支到本地当前分支
#1.时：
$ git checkout -b dev  origin/dev  #将远程新分支-创建到本地，比如dev

# 一般git push 前先进行 git pull ，
#如果no tracking information，表示本地和远程没有建立关联
$ git branch --set-upstream-to  dev  origin/dev     #本地分支dev和远程分支dev建立关联
$ git branch --set-upstream-to=origin/dev  dev


$ git push origin --delete master	#删除远程分支
$ git push origin :master # 等同于 $ git push origin --delete master
$ git push origin :master   #理解为 从本地push  空白到远程，即远程为空，删除
```



## 7、标签TAG

```shell
$ git tag   			 #查看所有标签
$ git tag [tag]			 # 新建一个tag在当前commit，例: git tag v1.0
$ git tag [tag] [commit] # 对指定的提交操作 打标签,例:git tag v0.5 f52c4s
$ git show [tag]   		 #查看tag信息
$ git tag  -a [tag] -m [message] [commit]  #创建带有说明的标签到指定commit版本
	#例如： git tag  -a v1.0 -m '周三版本1.0' 3fs3f5

$ git push origin 标签名    #提交指定tag 到远程origin仓库，git push origin v1.0
$ git push origin --tags    #一次性推送所有本地标签到远程origin仓库
$ git tag -d [tag]  #删除本地tag 如：git tag -d v1.0
$ git push origin :refs/tags/[tagName] # 删除远程tag

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

## 8、Git 设置代理

代理格式 `[protocol://][user[:password]@]proxyhost[:port]`
参考 https://git-scm.com/docs/git-config

设置 HTTP 代理：

```shell
$git config --global http.proxy http://127.0.0.1:8118
$git config --global https.proxy http://127.0.0.1:8118
```

设置 SOCKS5 代理：

```shell
$git config --global http.proxy socks5://127.0.0.1:1080
$git config --global https.proxy socks5://127.0.0.1:1080
```

Git 取消代理设置：

```shell
$git config --global --unset http.proxy
$git config --global --unset https.proxy
```

```shell
$git cat-file -t   414c3s8 #接一个hash值，查看哈希值的类型
$git cat-file -p   414sJ53 #查看哈希值内容
```













