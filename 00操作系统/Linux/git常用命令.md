# Git常用命令总结



## 1、git的基本概念

git有这么几个概念：

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310162457724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

## 2、初始化和配置
> git在使用前首先要配置user.name 和user.email 两个信息。这么做的目的可能是：
>
> - 记录每一个变更是谁在哪个时间点操作的
> - 在codereview时更方便自动给变更人发email
```shell
# 创建本地仓库
$ git init				# 在当前目录新建一个Git代码库
$ git init [project-name]# 新建一个目录，将其初始化为Git代码库

#作用域说明
$ git config --lobal  #lobal只对某一个仓库有效
$ git config --global #global对当前用户所有仓库有效
$ git config --system #system对系统所有登录的用户有效

#操作
$ git config --list		#显示当前的Git所有配置
$ git config --list  --global 	#查看global的配置信息，lobal同理
$ git config [--global] user.name "[name]"     #设置用户名
$ git config [--global] user.email "[email address]"   #设置email
$ git config --global user.name #查看global设置的用户名，email同理
```



## 3、本地操作（添加、提交、删除）

```shell
#添加到暂存区
$ git add [file1] [file2] ...   #添加指定文件到暂存区
$ git add [dir]   #添加指定目录到暂存区，包括子目录
$ git add .  #添加当前目录的所有文件到暂存区
$ git add * #等同git add .

# 提交本地仓库
$ git commit -m [message]  # 提交暂存区所有文件到本地仓库，并记录message信息
$ git commit [file1] [file2] ... -m [message]  # 提交暂存区的指定文件到仓库区

# 删除文件
$ git rm [file1] [file2] ...	# 删除工作区文件，并且将这次删除放入暂存区
$ git rm -r [文件夹]  #删除文件夹下所有
$ git rm --cached [file] 	# 停止追踪指定文件，但该文件会保留在工作区

# 重命名的快捷方式
$ git mv a.txt b.tx #将已经添加到暂存区的文件，重命名（相当于先重命名再提交暂存区）
```



## 4、查看信息

```shell
#查看当前仓库git状态
$ git status 

# 查看提交日志, 多个参数可以组合使用
$ git log			# 显示当前分支的版本历史
$ git log --all		# 显示所有日志
$ git log --praph	# 有些图形化的方式现实
$ git log --oneline #单行显示所有提交历史
$ git log --pretty=oneline  # ---pretty可以做一些格式化，接一些其他属性
$ git log --pretty=oneline  #和直接--oneline对比，就是显示完整的commit id,另者显示简短id
$ git log -5  --oneline 	# 单行显示过去5次所有提交

# 统计用户提交日志
$ git shortlog 		# 显示所有提交过的用户及各自提交的log信息
$ git shortlog -s	# 显示所有提交过的用户及提交次数
$ git shortlog -sn	# 显示所有提交过的用户，按提交次数倒序排

# 追踪责任，查看每一行的版本情况
$ git blame [file]	# 显示指定文件每一行是谁，在什么时间修改，提交情况（显示文件的每一行的版本情况）

#显示当前分支的最近几次提交
$ git reflog	
```

## 5、文件对比 git diff

```shell
#工作区 vs 暂存区
$ git diff	#比较工作区和暂存区的文件内容区别，(如果暂存区为空，已提交，那就是工作区和最新commit的区别，等同git diff HEAD)

#工作区 vs 版本库
$ git diff HEAD			 #显示工作区与当前分支最新commit之间的差异（工作区和本地仓库区别）
$ git diff HEAD -- [file]# 显示工作区某一个文件与当前分支最新commit之间的差异（工作区和本地仓库区别，命令有空格）

#暂存区 vs 版本库
$ git diff --cached [file]	#显示暂存区和上一个commit的差异（暂存区和本地仓库区别）

#多个版本commit比较
$ git diff [first-branch]...[second-branch] 

#显示某次提交和上一个commit的内容变化。
$ git show [commit版本号]	
```

## 6、分支管理

**查看分支**

```shell
$ git branch	# 列出所有本地分支
$ git branch -r # 列出所有远程分支
$ git branch -a	# 列出所有本地分支和远程分支
$ git branch -v# 查看分支的最后一次提交
$ git branch -av #列出所有分支并查看最后一次提交
```

**新建、切换分支**

```shell
$ git branch [branch-name]	# 新建一个分支，但依然停留在当前分支
$ git checkout [branch-name] # 切换到指定分支
$ git switch [branch-name] # 切换到指定分支--新版本
$ git checkout -b [branch]	# 新建一个分支，并切换到该分支
$ git switch -c [branch]	# 新建一个分支，并切换到该分支--新版本有 
```

**删除本地分支**

```shell
$ git branch -d [branch-name1] [branch-name2]	# 删除分支（本地的）可多个; 可以把-d 换为-D
```

**删除远程分支**

```shell
$ git push origin --delete [branch-name]  #两种方式都可以
$ git push origin :[branch-name]
```

**合并分支**

```shell
# 合并指定分支到当前分支
$ git merge [branch] 
# 合并指定分支到当前分支，会增加一条新的commit
$ git merge [branch] -m '信息'
$ git merge --no-ff  -m '日志' [branch]  //使用 --no-ff 方式合并,在删除分支后会保留提交信息，效果和上面

#选择一个commit，合并进当前分支
#例如：当在master修复一个bug时，dev分支当然也有这个bug，只需将修复bug那次commit 合并到dev分支，修复bug
$ git cherry-pick [commit]	

#查看日志分支提交记录，分支合并图
$ git log --graph --pretty=oneline   
```

**Fast-forward 方式和  NO-FF的区别？** 

以 dev分支合并到 master分支为例：

当两个分支合并时，如果节点明确，不如从master建立分支bug,然后修改完合并到master，会以fast-forward快进方式合并分支，也就是HEAD指针直接指向bug最新的commit作为master分支最新的commit，只是一个指针的移动，并没有去做提交。

 `git merge --no-ff -m '日志'`   方式合并，是非快进方式，就是在合并时会重新在master分支上新增一个commit，然后HEAD 指向它。  还有一个区别是 fast-forward 方式合并，删除分之后，不会看到分支的提交信息。而非快进方式合并可以。

## 7、撤销、回退、修改

**暂存区----->工作区**

- 放弃工作区，使用暂存区覆盖掉工作区的变更，例如：改错了但未添加到暂存区，需要回退一下工作区

```shell
$ git checkout -- [file]	# 恢复暂存区的指定文件到工作区,如果该文件修改后还没有提暂存区，那就会回退到和版本库一致（一定要加-- 有空格，不然的话checkout就是切换分支了）
$ git checkout .	# 恢复暂存区的所有文件到工作区
```

**版本库----->暂存区**

- 放弃暂存区。  例如：改错了，但是添加暂存区，未提交版本库，可以放弃暂存区，然后工作区修改完重新add ，或者使用 `git checkout -- [file] ` 直接放弃工作区的修改。

```shell
$ git reset head [file] #重置暂存区的文件，与当前版本库一致
```

**版本库----->重置暂存区 、工作区**

- 例如：改错了，已经提交到版本库了，暂未提交远程仓库这时候：就使用 `git reset --hard head^ ` 将版本库回退到上一版本，并且同时会覆盖工作区、暂存区 ，重新写。

```shell
$ git reset --hard  head^	# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard  head~1	# 重置暂存区与工作区，与上一次commit保持一致，两种写法^^^等同于~3
$ git reset --hard  [commitid]# 回滚到指定commit，重置暂存区和工作区
```

**git stash保存工作区**

- 例如：dev分支代码写到一半，这时候要切换到master分支，但是又不想commit，就可以使用`git stash`先保存工作区的修改，然后切换分支。最后回到dev分支，再恢复之前保存的工作区修改。

```shell
# 当在本分支没写完不能提交时，要切换分支，可以使用stash  先保存
$ git stash    		#保存工作区，然后去其他分支改bug
$ git stash list   	#查看存储记录，可以多次git stash就有多条记录
$ git stash apply  [可以是指定的]   #恢复工作区到上一次保存状态，或者指定某一次
$ git stash pop  [可以是指定的] #恢复工作区到上一次保存状态，或者指定某一次, 并删除stash记录
```


## 8、远程仓库、多人协作

```shell
#常用命令
$ git clone    		 # 克隆项目到本地
$ git fetch [remote] # 下载远程仓库的所有变动
$ git remote   		 #查看远程库信息
$ git remote  -v 	 #查看详细远程仓库信息 
$ git pull    	#下拉远程分支到本地当前分支
# 一般git push 前先进行 git pull，先合并，解决冲突
$ git push origin master   #推送本地master到远程分支master
$ git push origin dev      #推送本地dev  到远程dev分支（名字自己取）

#将远程分支dev ,创建到本地并命名为dev关联
$ git checkout -b dev  origin/dev  

#如果push还没有远程分支
$ git push --set-upstream-to origin dev #创建远程分支dev并将当前分支push到远程分支

#如果no tracking information，表示本地和远程没有建立关联
$ git branch --set-upstream-to  dev  origin/dev     #本地分支dev和远程分支dev建立关联
$ git branch --set-upstream-to=origin/dev  dev

#删除远程分支
$ git push origin --delete dev	
$ git push origin :dev # 等同于 $ git push origin --delete master，理解为 从本地push空白到远程，即远程为空，删除
```
## 9、标签TAG
```shell
$ git tag   			 #查看所有标签
$ git tag [tag]			 # 新建一个tag在当前commit，例: git tag v1.0
$ git tag [tag] [commit] # 对指定的提交操作 打标签,例:git tag v0.5 f52c4s
$ git show [tag]   		 #查看tag信息
$ git tag  -a [tag] -m [message] [commit]  #创建带有说明的标签到指定commit版本
	#例如： git tag  -a v1.0 -m '周三版本1.0' 3fs3f5
$ git tag -d [tag]  #删除本地tag 如：git tag -d v1.0

$ git push origin 标签名    #提交指定tag 到远程origin仓库，git push origin v1.0
$ git push origin --tags    #一次性推送所有本地标签到远程origin仓库
$ git push origin :refs/tags/[tagName] # 删除远程tag

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```
## 10、变基 rebase
### 10.1修改以及合并commit

**修改最新commit的message**

```shell
#修改最后一次提交（例如：提交错了，或者提交信息写错了，可以修正）
$ git commit --amend #如果这时候暂存区没修改，相当于是直接编辑上一次提交说明
$ git commit -m 'first'
$ git add  a.txt
$ git commit --amend -m 'hello'# 这三条命令只会产生一次提交，--amend就是对上次提交的修正 
```

**修改历史commit的message**

```shell
#修改历史commit的日志message
$ git rebase -i [要求改的commit的上一个commitid] #注意是上一个commitid
#这时候会进入一个编辑页
pick a8e9b43 my message（日志信息）  # 将pick改为reword ,然后保存，
#这时候又会进入一个新的编辑页，用来修改这次a8e9b43 的提交日志
my message   # 将这个日志修改保存即可
```

**将多个commit合成一个commit**

使用git rebase -i  交互的变基，使用squash方式，使用这个commit，但是要合并到之前的commit


```shell
#比如，git log --oneline 看一下，有6个commit，想把 12345合成一个commit
$ git log --oneline 
f6d07f7 (HEAD -> demo) commit5
ed91cf3 commit4
06cca74 commit3
cc417da commit2
43278d7 commit1
151a7d0 helloworld
$ git rebase -i 151a7d0 #一定要选他们的父亲的commitid,会进入编辑页
pick 43278d7 commit1
pick cc417da commit2
pick 06cca74 commit3
pick ed91cf3 commit4
pick f6d07f7 commit5

#使用squash方式，squash <commit> = use commit, but meld into previous commit
#注意：使用squash，顺序很重要，必须是把下面的commit合并到上面的commit，也就是squash上面必须有一个pick的commit
#例如： 我要把1 2 3 4 合并到 5
pick 43278d7 commit1
s cc417da commit2
s 06cca74 commit3
s ed91cf3 commit4
s f6d07f7 commit5
#例如： 我要把 1 2 合并到5,但是3 4 不变 
pick 06cca74 commit3
pick ed91cf3 commit4
pick f6d07f7 commit5
s 43278d7 commit1
s cc417da commit2
#只要注意顺序即可。保存后会进入另一个编辑页，我们可以写入最终合并的commit的message，然后保存即可

```
```shell
rebase的修改模式
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.

```

### 10.2 分支rebase

```shell
#在有远程分支的情况下，如果分支合并弄得log比较乱，可以直接git rebase 使分支的log成为一条线
$ git rebase  #或 git rebase origin

#本地分支rebase，比如本地有三个分支--master 生成的server ， server生成的client
$ git checkout server
$ git rebase  master   #先切到server分支，然后将server和master共同节点之后的所有更改，在master分支上重新来一次，也就是变基。最后server分支的HEAD指向它（这时候其实server已经和使用merge合并后的代码一样了，但是master分支还是原来的代码）
$ git checkout master #切换到master
$ git merge server  #快速合并，其实就是HEAD指向编辑后的最新commit，和server分支的HEAD指向同一个commit
//这样操作的结果，和git merge 是一样的，区别在于，他们的日志是一条直线

#将一部分的变更，变基到另一个分支的操作，比如：我指向把client分支上的操作合并到master分支，但是由于client是从server上生出来的，并不想把server分支的一些操作合并到master，怎么办？
$ git  rebase --onto master  server client #--onto就是指定master为基准分支，把client相对于server分支的改变合并到master分支上，但是不包括server分支相对于master的改变。
# 这时候其实就是client 指向了master变基后的commit，
$ git checkout master 
$ git merge client  # 就相当于master和client分支的部分代码（不包含server分支）合并了。

#合并server 到master分支
$ git rebase master server  #把server分支变基到 master上
$ git checkout master #切换到master分支
$ git merge server #快速合并两个分支

//总之最后和git merge的结果基本一样，只是日志的方式不同
```

分支的rebase详解参考：

[http://iissnan.com/progit/html/zh/ch3_6.html](http://iissnan.com/progit/html/zh/ch3_6.html)




## 11、分离头指针情况

**什么是分离头指针 detached HEAD？** 

当我们执行 `git checkout [commitid]` 将我们某一次的commit 给checkout 出来，就是分离头指针状态。

其实也就是我们工作在在一个没有分支的状态，head指向的是一个commit，而不是一个分支。当然我们可以继续在这个上面继续提交代码，但是一旦我们切换分支后，我们就切不回来了，因为它没有分支。而我们做的提交可能会被git当做垃圾给清除掉。除非我们为它创建一个新分支。

**但是分离头指针什么时候用呢？**

比如：我们要临时做个实验性的修改，这个时候我们就可以使用分离头指针状态，然后切换回原来分支就不用管了，不保存。

当然如果想继续用那个分离头指针的状态，可以为他创建一个分支`git branch <new-branch-name>  <commitid>` 。

还有： 我们在使用`rebase`命令修改 message时，其实就是利用了分离头指针状态。

**head 的指向：**

我们的HEAD可以指向一个分支，也可以指向一个 commit， 而分支的内容和commit的内容其实都是一串hash值，这穿hash值是一个commit类型，也就是 HEAD其实就是指向了一个commit类型，分支其实就是它最新的一次commit。

## 12、git代理设置

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

## 13. git原理—commit、tree、blob

先学会两个命令，`git cat-file -t`  和 `git cat-file -p` ，一个看类型，一个看内容

```shell
$git cat-file -t   414c3s8 #接一个hash值，查看哈希值的类型
$git cat-file -p   414sJ53 #查看哈希值内容
```

当我们研究.git 文件夹时很有用。

.git 文件夹下有这么几个文件夹和文件需要知道：HEAD，config，refs文件夹，objects文件夹

```shell
.git
|-HEAD       --------内容：ref: refs/heads/master，就是指向当前是哪个分支
|-config	 --------配置文件，我们git config --local 所做的操作都写在这个文件
|-refs	（文件夹）		
	 |-heads（文件夹）	 -----存放分支的hash,有几个分支，就会有几个文件在该文件夹下，文件的内容是个hash值
	 |-remotes（文件夹）  -----存放远程分支的hash
	 |-tags（文件夹）     -----存放标签
	 |-stash   			-----存放git stash所做的工作区暂存的hash

|-objects（文件夹）  -------存放的是git所有操作所对应的文件快照，以哈希值前两位为文件夹，子集存的是以剩余hash值命名的文件
	 |-12
	 |-4a
```

**git 存储文件的规则**： 只要是文件内容相同，git就认为是同一个文件，不管文件名是什么。

**git 的三种对象**： **commit**  、**tree**  、**blob** 

借用一张图看下（网上找的）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310161440835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

试验一下：创建一个git仓库，只创建一个文件夹text,然后在文件夹下创建一个文件test.txt ，文件内容写一句话“这是一个测试文件” ，commit 到本地仓库，然后挨个看一下git的这三个对象

```shell
#此时的文件夹结构
|-.git       
|-text
	 |test.txt
```

```shell
#查看本次提交的commitID
$ git log --oneline
1ccbf8c (HEAD -> master) first commit

#查看这个1ccbf8c 是个什么东西？
$ git cat-file -t 1ccbf8c  #输出了，是个commit类型
commit

#查看这个东西内容是什么
$ git cat-file -p 1ccbf8c    #内容是 tree 、author和 committer
tree ce732ad8e57dd5b0b9018f0f11a5984d36e01fe1
author answer <962889441@qq.com> 1615362590 +0800
committer answer <962889441@qq.com> 1615362590 +0800

first commit

#看一下这个tree里面是什么？
$ git cat-file -p ce732ad8e5   #里面也是一个tree类型，对应text文件夹
040000 tree dbe740631af037c561a4b479528962efed64941f    text

#继续看这个tree的内容
$ git cat-file -p  dbe740631a    #里面是个blob,也就是我们的test.txt文件
100644 blob 1a237b67ce0f08a91a764a66f72e15e691191af8    test.txt

#继续看下blob的内容
$ git cat-file -p 1a237b67ce0     #这里输出的是我在test.txt里面写的一句话，也就是文件的内容
这是一个测试文件


```

总结一下就是：

- 我们的.git文件夹的HEAD 文件夹存的是指向了master分支
- 然后 .git/refs/heads/master 文件存的内容就是我们master分支最后一次提交的hash值，也就是 1ccbf8cb34f60186cc39cd5c7481d84fd748e9f9 
- 然后这次commit 存的是tree(1)、parent、author、committer等
- tree(1) 存的是tree(2) ，也就是text文件夹
- tree(2)内容是 blob,也就是我们的test.txt文件
- 最后这个blob也就是我们文件的真实内容

 当我们一层层起剥开这个hash值的内容的时候就发现：
我们的三个对象，commit 包含tree 和blob ，tree也可以包含tree和blob 。   blob 就是最后存储的文件快照。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310161417536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70) 

git的对象在.git/objects文件夹下存放,都是以hash值前两位为文件夹名称：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310162133143.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)