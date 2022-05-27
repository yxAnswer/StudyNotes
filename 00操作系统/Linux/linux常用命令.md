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

```shell
ls 		#列出目录内容
ls -a 	#列出所有文件和目录，包括隐藏的
ls -l	#列出详细格式的目录
ll 		#ls -l  的快捷方式，相同
ls -t	#用文件和目录的更改时间排序
ls -r	#反向排序
ll /home/     #显示指定目录下的内容
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

**在底行模式操作：**

```shell
:wq    #保存并退出
:q!   #不保存，强制退出
:w #保存不退出
:q #退出不保存
set nu #显示行号
set nonu #不显示行号。

:! #可以再编辑文本的时候，临时输入命令，比如我们查看ip,:! ip addr，找到ip,然后再回到编辑页面。

:/ #可以进行查找，比如`:/port` 可以列出所有查找到port的文本，按字母n,可以跳转到下一个查找的记录。shift+n 往上找下一个查找记录。

:3,5s/test/test2/g  #:表示，将3 到5行，test替换成test2 ，g标识如果有多个批量替换。 比如将10-15行的所有java替换成go， `:10,15s/java/go/g`

:%s/word1/word2/g #从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2
n1,n2s/word1/word2/g #n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代
为 word2
```

**在命令模式操作：**

```shell
 hjkl #移动光标：h、j、k、l  分别对应，左、下、上、右

 ^ $ #^移动到一行的行首，$ 移动到行尾部  也就是shift+6 和shift+4

 gg  #移动到文档第一行行首
 G   #移动到文档最后一行行首
 10G #移动到第10行，11G移动到11行。
 x   #删除内容，删除一个字符
 dd  #删除游标所在的那一整行
 u   #复原原来的操作
 v   #选中范围按y即复制
 p   #粘贴
```

**如何复制粘贴？**

- 按esc进入命令模式
- yy  复制一行，p 粘贴。 y$ 复制光标到行位。 3y 复制3行，5y复制5行
- dd 剪切一行，类似。 u为撤销指令。
- 按v 进入可视模式，移动光标选中要复制的内容
- 按y 复制命令进行复制， d是剪切命令
- 按 p 复制命令进行复制（put）

**可视模式：**

```
v #可以模式可以按照字符选择
shift+v  #行模式，可以按行选择，进行删除、复制、粘贴等
ctrl+v  #块模式，可以按照块选择，进行操作
```



## 5、压缩/解压

### 5.1 tar 命令

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

### 5.2 使用zip/unzip 压缩解压：

```shell
#安装zip/unzip
yum install zip
yum install unzip

zip -r mydata.zip mydata #把mydata目录压缩为mydata.zip
unzip mydata.zip -d mydatabak #把mydata.zip解压到mydatabak目录里面
zip -r abc123.zip abc 123.txt #把abc文件夹和123.txt压缩成为abc123.zip
unzip wwwroot.zip #把wwwroot.zip直接解压到/home目录里面
unzip abc\*.zip #把abc12.zip、abc23.zip、abc34.zip同时解压到/home目录里面
unzip -v wwwroot.zip # 查看wwwroot.zip里面的内容
unzip -t wwwroot.zip #验证wwwroot.zip是否完整 
unzip -j wwwroot.zip #把wwwroot.zip里面的所有文件解压到第一级目录
```

**主要参数**

```
-c：将解压缩的结果
-l：显示压缩文件内所包含的文件
-p：与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换
-t：检查压缩文件是否正确
-u：与-f参数类似，但是除了更新现有的文件外，也会将压缩文件中的其它文件解压缩到目录中
-v：执行是时显示详细的信息
-z：仅显示压缩文件的备注文字
-a：对文本文件进行必要的字符转换
-b：不要对文本文件进行字符转换
-C：压缩文件中的文件名称区分大小写
-j：不处理压缩文件中原有的目录路径
-L：将压缩文件中的全部文件名改为小写
-M：将输出结果送到``more``程序处理
-n：解压缩时不要覆盖原有的文件
-o：不必先询问用户，unzip执行后覆盖原有文件
-P<密码>：使用zip的密码选项
-q：执行时不显示任何信息
-s：将文件名中的空白字符转换为底线字符
-V：保留VMS的文件版本信息
-X：解压缩时同时回存文件原来的UID/GID
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

```shell
[root@localhost answer]# kill -l     //有64种方式
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

常用 `kill -9  pid` 强制杀死

```shell
1) SIGHUP       终端的控制进程结束，通知session内的各个作业，脱离关系 
2) SIGINT       程序终止信号（Ctrl+c）
3) SIGQUIT      和2号信号类似（Ctrl+\），产生core文件
4) SIGILL       执行了非法指令，可执行文件本身出现错误 
5) SIGTRAP      有断点指令或其他trap指令产生，有debugger使用
6) SIGABRT      调用abort函数生成的信号 
7) SIGBUS       非法地址（内存地址对齐出错）
8) SIGFPE       致命的算术错误（浮点数运算，溢出，及除数为0 错误）
9) SIGKILL      用来立即结束程序的运行（不能为阻塞，处理，忽略）
10) SIGUSR1     用户使用 
11) SIGSEGV     访问内存错误
12) SIGUSR2     用户使用
13) SIGPIPE     管道破裂
14) SIGALRM     时钟定时信号
15) SIGTERM     程序结束信号（可被阻塞，处理）
16) SIGSTKFLT   协处理器栈堆错误
17) SIGCHLD     子进程结束，父进程收到这个信号并进行处理，（wait也可以）否则僵尸进程
18) SIGCONT     让一个停止的进程继续执行（不能被阻塞）
19) SIGSTOP     让一个进程停止执行（不能被阻塞，处理，忽略）
20) SIGTSTP     停止进程的运行（可以被处理和忽略）
21) SIGTTIN     当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行.
22) SIGTTOU     类似SIGTTIN,但在写终端时收到
23) SIGURG      有紧急数据或者out—of—band 数据到达socket时产生
24) SIGXCPU     超过CPU资源限定，这个限定可改变
25) SIGXFSZ     当进程企图扩大文件以至于超过文件大小资源限制
26) SIGVTALRM   虚拟时钟信号（计算的是该进程占用的CPU时间）
27) SIGPROF     时钟信号（进程用的CPU时间及系统调用时间）
28) SIGWINCH    窗口大小改变时发出
29) SIGIO       文件描述符准备就绪，可以进行读写操作
30) SIGPWR      power failure
31) SIGSYS      非法的系统调用

```

### 查看ip及网卡信息

#### 1）ifconfig  查看网卡信息

如果出现centos7未找到 ifconfig命令

```shell
ls /sbin/ifconfig # 查看是否环境变量没有ifconfig引起
yum search ifconfig # 如果没有安装，查看匹配ifconfig的包
========================================================= 匹配：ifconfig =============================
net-tools.x86_64 : Basic networking tools

yum install install net-tools.x86_64  # 根据搜索结果，安装即可
ifconfig #查看
```



#### 2）ip addr 也可以查看ip信息

### ping 查看网络连接情况

### netstat -an  查看端口占用情况

功能说明：**查看网络端口的使用情况**
举 例：`netstat -tunlp | grep nginx`
`-t` ：显示tcp端口
`-u` ：显示UDP端口
`-n `：指明拒绝显示别名
`-l` ：指明listen的
`-p` ：指明显示建立相关连接的程序名
安装netstat命令：`yum -y install net-tools  `

`netstat -lnp|grep 9000 #查看端口占用`

### top命令

**功能说明：监控Linux系统状况，比如cpu、内存的使用**
举 例：按住键盘q退出  

top -s   :使top命令在安全模式中运行。

E 可以切换 内存显示单位 kb Mb gb等

s ：小写s可以制定刷新延迟时间，刷新频率

M: 根据主流内存大小进行排序

P: 根据cpu 百分比大小排序

### du   df命令

**功能说明：统计大小** 

> `df -h`查看系统中文件的使用情况
>
> ##### `du -sh *`查看当前目录下各个文件及目录占用空间大小
>
> du -s * |sort -nr |head 选出排在前面的10个
>
> 



### uniq命令

**功能说明：对排序好的内容进行统计**
举 例：uniq -c 123.txt | sort -n  

可以查出内容出现次数、并且通过sort 排序

### cal 命令

**功能说明：查看日历**
举 例：cal 2008   查看2008年日历

cal --help 可以查看cal其他用法

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



**用户组**

```shell
chown mysql:mysql -R /usr/local/mysql  # 比如将mysql用户给mysql用户组， 赋给文件夹/usr/local/mysql权限
```



## 常见问题

```shell
# Unit firewalld.service could not be found  没有安装防火墙
Centos7 下默认的防火墙是 Firewall，替代了之前的 iptables，Firewall 有图形界面管理和命令行管理两种方式，本文简要介绍命令 行Firewall 的使用。

#如果提示：Unit firewalld.service could not be found. 说明防火墙没有安装，需要安装

yum install firewalld firewall-config
3重启、关闭、开启firewalld.service服务
service firewalld restart 重启
service firewalld start 开启
service firewalld stop 关闭

#添加自定义端口
firewall-cmd --zone=public --permanent --add-port=8010/tcp

#查看firewall服务状态
systemctl status firewalld.service 

#查看firewall的状态
firewall-cmd --state
#查看防火墙规则
firewall-cmd --list-all 
#添加服务
firewall-cmd --permanent --zone=public --add-service=http

#使最新的防火墙规则生效
firewall-cmd --reload
```





