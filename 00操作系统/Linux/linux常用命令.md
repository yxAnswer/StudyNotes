# linux常用命令

[TOC]

## 1、linux 目录结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326154835771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

借用一张图，Linux 中，一切皆文件，所以，文件的根目录为/    

centos 系统cd / 到根目录，ls ,查看所有文件如下：

```
bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root
run  sbin  srv  sys  tmp  usr  var
```

这么多目录，我们又不是运维没必要都知道。主要的目录为

/ : 这个是系统的根目录，一般只存放目录，不存放文件

/bin : /user/bin:   (binaries)存放二进制可执行文件。比如（ls,tar,mc,cat等）

/boot: 放置的是Linux系统系统时用到的一些文件

/dev :（devices）设备，即存放Linux系统下的设备文件，比如光驱。

**/etc**: （etcetera）存放系统配置文件，这个目录很重要，会经常用到。

**/home**: 系统默认的用户目录，除了root用户，其他用户都会再此目录下。比如test这个账号就会再home下生成一个test 目录

/lib:  (library) :存放系统使用的函数库。比较重要的比如：/lib.modules

**/root**: 系统管理员root这个账号的主目录，和home一个意思。

/sbin：(super user binaries)放置的是系统管理员使用过的可执行命令，一般用户只能查看不能设置和使用。

**/usr**: (unix shared resources) 应用程序存放目录，这个目录我们经常用到。

/usr/bin 存放应用程序。

/usr/share  存放共享数据，

/usr/lib 存放不能直接运行的，却是很多程序运行所必须的一些函数库文件。

/usr/local:存放软件升级包。我们软件一般装在这，比如mysql ,tomcat

/usr/sare/doc ：系统说明文件存放目录。

/usr/share/man:程序说明文件存放目录，使用 man ls 时会查询/usr/share/man/man1/ls.1.gz的内容建议单独分区，设置较大的磁盘空间。

/var:(variable) 放置系统执行过程中经常变化的文件，比如/var/log:日志文件；/var/log/message：所有的登录文件存放目录；/var/spool/mail:邮件存放的目录；/var/run:程序或服务启动

**/opt**: 给主机额外安装软件的目录。比如一直用/usr/local目录，现在可以装在/opt目录。看喜好

/tmp:(temporary)：临时文件

/srv:服务启动之后需要访问的数据目录。比如www服务需要访问的网页数据存放在/srv/www内

/mnt:  挂载点存放

/proc: 此目录的数据都在内存中，如系统核心，外部设备，网络状态等

## 2、目录操作

### ls 命令：列出

```
ls 		列出目录内容
ls -a 	列出所有文件和目录，包括隐藏的
ls -l	列出详细格式的别彪
ll 		ls -l  的快捷方式，相同
ls -t	用文件和目录的更改时间排序
ls -r	反向排序
ll /home/     显示指定目录下的内容
```

### cd 命令:切换目录

```
cd ~		切换到用户的主目录，root是到root，其他账号到home
cd /		切换到根目录
cd ..		切换到上一级目录
cd /usr/local	切换到指定目录
```

### pwd 命令：显示当前工作目录

`pwd`

### mkdir 命令： 创建文件夹

```
mkdir  /usr/local/mysql    在指定目录下创建文件夹
```

### find 命令：查找目录

```
find /root -name '*test*'  查看/root目录下的 名称中包含 test的目录。
```

### mv 命令： 修改、移动、剪切命令

**mv语法不仅可以对目录进行剪切，重命名操作，对文件和压缩包等都可执行剪切，重命名操作**

```
mv oldFolder newFolder	修改文件夹名称
mv oldFile newFile  修改文件的名称
mv oldFile  /usr/local  将文件oldfile移动到/usr/local目录
mv oldFolder /usr/local 将目录移动到/usr/local目
```

### cp 命令：拷贝命令copy

```
cp -r oldFolder /usr/local  将oldFolder拷贝到新目录下，-r 表示递归
cp oldFile  /usr/local  将文件拷贝到指定目录
```

### rm 命令：删除命令

rm -rf   可以强制删除任何目录和文件

```
rm -r  文件名或文件夹名     ：删除并询问是否删除
rm -rf 名称				：强制删除，不询问
```

## 4、文件操作

### touch 命令：创建文件

```
touch   aaa.txt  创建文件aaa.txt
touch  /usr/local/a.txt   在指定目录下创建文件
```

### cat、more、less、tail :都可以查看文件

区别：

**cat** :在控制台只能查看最后一屏，但是远程连接终端有滚动条就没有限制了，常用。

**more**:可以显示百分比，回车查看下一行，空格查看下一页，q退出查看

**less**:和more差不多，q退出

**tail**: 可以指定查询的行数，tail -10 :查看文件后10行，ctrl+c 结束查看

**可以使用tail -f   循环读取命令对文件进行动态监控，比如日志文件**

### vim 命令：编辑

```
vim  aa.txt		编辑aa.txt文件，进入vim编辑器
```

vim有三种模式：**命令模式**、**编辑模式**、**底行模式**

执行vim 命令进入的是命令模式，此时是不能编辑的，输入 a或i 或o  ,可以进入编辑模式。

编辑模式：a ,会从当前光标的后面位置开始输入，i 会从当前光标的前面开始输入。 o会另起一行进行输入。

底行模式： 按键盘的 Esc 键进入退出编辑模式，输入 ：冒号进入底行模式

:wq    保存并退出

:q!   不保存，强制退出

:w 保存不退出

:q 退出不保存

## 5、压缩/解压

linux中的打包文件一般以.tar结尾的，压缩文件一般以.gz结尾。

打包并压缩会以.tar.gz结尾。

**压缩命令：tar -zcvf   打包压缩后的文件名     要打包的文件**   ：打包并压缩指定文件并命名。

```
tar -zcvf   aaa.tar.gz    a.txt b.txt c.txt
其中
	-z 调用gzip压缩命令进行压缩
	-c 打包文件
	-v 显示运行过程
	-f 指定文件名
tar -zcvf xxx.tar.gz /test/*  打包压缩整个文件夹下的
```

**解压命令：tar -xvf   压缩文件**

```
tar -xvf xxx.tar.gz
其中：
	x：代表解压
-c :表示指定目录
tar -xvf xxx.tar.gz -c /usr/local    解压到指定目录
```



## 6、其他常用命令

### grep：搜索命令

grep  要搜素的字符串   要搜索的文件 ，比如grep  to  /usr/sudo.conf

grep to /usr/sudo.conf  --color     搜出的to 高亮

### ps -ef  查看系统进程

ps -ef :查看当前系统中运行的进程

### |  管道命令 

将前一个命令的输出作为本次目录的输入。

比如：ps -ef | grep system    将所有进程信息作为搜索system 字符串的资源进行搜索

### kill  -   杀死进程

kill  -进程pid    ,比如 kill -10

### ifconfig  查看网卡信息

### ping 查看网络连接情况

### netstat -an  查看端口占用情况



## 7、linux 下的权限命令

linux下是一个多用户的系统，每个文件、目录都有权限。执行`ls -l`

```
[root@iZszxghs0ozok0Z usr]# ls -l
total 92
dr-xr-xr-x.  2 root root 24576 Mar 23 01:39 bin
drwxr-xr-x.  2 root root  4096 Apr 11  2018 etc
drwxr-xr-x.  2 root root  4096 Apr 11  2018 games
drwxr-xr-x. 34 root root  4096 Mar 23 01:06 include
dr-xr-xr-x. 30 root root  4096 Mar 23 01:33 lib
dr-xr-xr-x. 37 root root 20480 Mar 23 01:39 lib64
drwxr-xr-x. 21 root root  4096 Mar 23 01:33 libexec
drwxr-xr-x. 14 root root  4096 Mar 23 01:04 local
dr-xr-xr-x.  2 root root 12288 Mar 23 01:39 sbin
drwxr-xr-x. 79 root root  4096 Mar 23 01:39 share
drwxr-xr-x.  4 root root  4096 Nov 29 11:34 src
lrwxrwxrwx.  1 root root    10 Nov 29 11:34 tmp -> ../var/tmp

```

可以看到，drwxr-xr-x  类似的东西，表示权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019032617160669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

如上图，linux权限用10位字符来表示。

第一位表示文件类型，d 目录，-文件 ，l 链接

剩下9位，每3位一组。

第234 位，表示所属用户权限。

第456位，表示所属组的权限。

第789，表示其他用户的权限。

linux下的用户，可以属于某个组，当然还有其他用户，这些关系的权限也由这控制。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326172144528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

r：代表权限是可读，r也可以用数字4表示

w：代表权限是可写，w也可以用数字2表示

x：代表权限是可执行，x也可以用数字1表示

### 修改文件/目录的权限的命令：chmod

```
//修改aaa.txt 文件的权限
chmod u=rwx,g=rw,o=r aaa.txt    // u用户权限，g组权限，o 其他用户权限
```

当然，上面说了，r,w,x可以用4,2,1 进行代替，那么下面命令可实现同样效果。

```
chmod 764 aaa.txt   
//u: 4,2,1 加起来是7 
//g： 4,2  加起来是6
//o: 4    也就是说，只有r 可读权限。
```











