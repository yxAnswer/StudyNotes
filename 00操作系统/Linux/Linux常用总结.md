# Linux 常用总结

[toc]

# 1、学习linux之  vmware相关

## 1.1vmware 3种网络模式说明

Vmware 虚拟机对应三种网络模式：

- VMnet0 虚拟交换机 ：Bridged桥接模式  

  > 特点：
  > a. 默认使用VMnet0，不提供DHCP服务（DHCP服务是指由服务器控制的一段IP地址范围，当客户机登录服务器时会
  > 自动获取服务器分配的IP地址与子网掩码）
  > b. 虚拟机与外部主机需要在同一个网段上，与局域网的其它机器没有区别。
  > c. 可以与局域网内其它主机通信，可以与外部网络通信
  > d. 容易与局域网其他主机引起ip地址冲突  

- VMnet1 虚拟交换机 ：Host-Only仅主机模式  

  > 特点：
  > a. 默认使用VMnet1，提供DHCP服务
  > b. 虚拟机可以和物理主机互相访问，但虚拟机无法访问外部网络  

- VMnet8 虚拟交换机 ：NAT模式  

  > 特点：
  > a. 默认使用VMnet8，提供DHCP服务
  > b. 虚拟机可以和物理主机互相访问，可访问外部网络
  > c. 局域网内其它机器访问不了  

## 1.2 vmware 安装centos7后的网络设置

- Bridged桥接模式
  重启主机的命令：reboot
  重启网卡的命令：systemctl restart network.service
  查看ip地址的命令:ip addr
  ping命令可以检测网络是否畅通：ping ip地址
  结束ping命令：ctrl + c
  安装ctrl +l 可以清屏
  可以访问外网
  容易与局域网的其它机器ip地址冲突
- Host-Only仅主机模式
  一般情况下不能访问外网
  不会与局域网的其它机器ip地址冲突
- NAT模式
  可以访问外网
  不会与局域网的其它机器ip地址冲突

当我们安装完，无法上网，ip addr 也看不到ip的时候，可能是网卡配置的问题：

```shell
cd /etc/sysconfig/network-scripts  #到此目录进行配置
#比如
[root@localhost network-scripts]# cat ifcfg-eno16777736 
HWADDR=00:0C:29:E1:26:E1
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=8df640e5-ae5b-4637-938a-f27ceabd0093
ONBOOT=yes

#设置固定ip 根据实际情况填写
#HWADDR=00:0C:29:51:F5:5F
IPADDR=192.168.0.106
NETMASK=255.255.255.0
NM_CONTROLLED=no
GATEWAY=192.168.0.1
DNS1=211.136.192.6
```



## 1.3 vmware虚拟机的备份与快照恢复

- 克隆后： `systemctl restart network.service`命令执行会报错，原因是MAC地址不正确

- 安装键盘的tab键可以对命令进行补全

- 网卡路径：`/etc/sysconfifig/network-scripts/ifcfg-eno16777728`

- 使用vi工具进行编辑网卡信息：`vi /etc/sysconfifig/network-scripts/ifcfg-eno16777728`（按住键盘的i进入编辑模式，按住键盘左上角esc键退出编辑模式，再输入：wq进行保存）

- 快照:相当于备份

# 2、linux常用操作

## 2.1 linux目录分类介绍 

```shell
[root@localhost /]# ls
bin  boot  CentOS-Base.repo  dev  env  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  user  usr  var
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326154835771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTExMzgxOTA=,size_16,color_FFFFFF,t_70)

- /：根目录，一般根目录下只存放目录，不要存放文件，也不要修改，或者删除目录下的内容

- /mnt：测试目录

- /root：root用户的家目录

- /home：普通用户的家目录

- /tmp：临时目录(比如文件上传时)

- /var：存放经常修改的数据，比如程序运行的日志文件

- /boot：存放的启动Linux 时使用的内核文件，包括连接文件以及镜像文件

- /etc：系统默认放置配置文件的地方

- /bin：所有用户都能执行的程序

- /sbin：只有root才能执行的程序

- /usr：用户自己的软件都可以放到这儿来

- /dev：存放硬件设备的地方(/dev/cdrom)

- /media：挂载光盘使用的

- 挂载光盘：mount /dev/cdrom /media

- 卸载光盘：umount /dev/cdrom绝对路径：说白了就是完整的路径

- 相对路径：相对于当前位置路径 ./ 代表的是当前目录的意思 ../ 代表的是上一级目录的意

## 2.2 工作中常用基础命令

### 2.2.1 man 命令

```shell
#功能：查看帮助文档
man ls  #将ls帮助文档按行打印 ，按q退出
ls --help #跟man类似，将ls说明文档全部打印
```

### 2.2.2 help 命令

```shell
#功能：查看内部命令帮助
[root@localhost /]# help if
if: if 命令; then 命令; [ elif 命令; then 命令; ]... [ else 命令; ] fi
    根据条件执行命令。
    
    `if COMMANDS'列表被执行。如果退出状态为零，则执行`then COMMANDS' 
    列表。否则按顺序执行每个 `elif COMMANDS'列表，并且如果它的退出状态为
    零，则执行对应的 `then COMMANDS' 列表并且 if 命令终止。否则如果存在的
    情况下，执行 `else COMMANDS'列表。整个结构的退出状态是最后一个执行
    的命令的状态，或者如果没有条件测试为真的话，为零。
    
    退出状态：
    返回最后一个执行的命令的状态。
```

### 2.2.3 cd 命令

```shell
#功能：切换目录
cd /		#切换到根目录
cd /usr/local #切换到制定目录 /usr/local目录下
cd .. 		#返回上一层目录
cd - 		#放回上一次跳转前的目录，比如从/ cd到/usr/local下，cd -就会返回 /
cd ~  		#切换到用户的主目录，root是到root，其他账号到home
```

### 2.2.4 ls 命令

```shell
#功能：列出目录内容
ls 		#列出目录内容
ls -a 	#列出所有文件和目录，包括隐藏的
ls -l	#列出详细格式的目录
ll 		#ls -l的快捷方式，效果相同
ls -t	#按照更改时间 倒叙
ls -r	#反向排序
ll /home/     #显示指定目录下的内容
#多个属性可以合并使用比如：
ls -alrt # 列出目录下所有包括隐藏的、详细格式、按时间、正序
ll -t #列出详细内容，按时间倒叙-常用
```

### 2.2.5 pwd 命令

```shell
#功能：查看当前所在的目录
[root@localhost home]# pwd
/home
```

### 2.2.6 cat 命令

```shell
#功能：查看文件内容（一般用于小文件，全部打印出）
cat a.txt #查看a.txt内容
```

### 2.2.7 more 命令

```shell
#功能：查看文件内容（用于查询大文件内容）
more 文件名 #按住空格，进行翻页； 按回车进行一行行的读取； 按q 退出浏览
```

### 2.2.8 head 命令

```shell
#功能：查看文件最前面的N 行
head -20  nginx.conf #查看文件的前20行
```

### 2.2.9 tail 命令

```shell
#功能：查看文件后面N行，一般用于查看日志，-f不断刷新
tail -f log.txt #持续刷新查看日志最后10行
tail -20f log.txt#持续刷新日志最后20行
```

### 2.3.0 less 命令

```shell
#功能：查看大文件内容，和more基本想同
less  file.txt #查看文件
```

### 文件查看: 小结

**cat** :在控制台只能查看最后一屏，但是远程连接终端有滚动条就没有限制了，常用。

**more**:可以显示百分比，回车查看下一行，空格查看下一页，q退出查看

**tail**: 可以指定查询的行数，tail -10 :查看文件后10行，ctrl+c 结束查看

**可以使用tail -f   循环读取命令对文件进行动态监控，比如日志文件**

**less**:和**more** 基本想同，区别如下：

**more和less的区别：**

-  less可以按键盘上下方向键显示上下内容,more不能通过上下方向键控制显示
-  less不必读整个文件，加载速度会比more更快
-  less退出后shell不会留下刚显示的内容,而more退出后会在shell上留下刚显示的内容

### 2.3.1 touch 命令

```shell
#功能：创建一个空文件
touch a.txt #创建名为a.txt的空文件
```

### 2.3.2 mkdir 命令

```shell
#功能：创建一个目录
mkdir test#创建一个test目录
mkdir -p test/demo1/demo2 #递归创建文件夹，如果不加-p会报错，比如

[root@localhost env]# mkdir test/demo1/demo2   
mkdir: 无法创建目录"test/demo1/demo2": 没有那个文件或目录

#目录和文件的区别
[root@localhost env]# ll
总用量 4
-rw-r--r--. 1 root root 15 5月  27 18:25 a.txt
drwxr-xr-x. 2 root root  6 5月  27 18:43 test
#第一位：d表示目录，-表示文件
```

### 2.3.3 rmdir 命令

```shell
#功能：删除空目录（只能用于删除空目录）
rmdir test/demo1/demo2 #删除制定路径的目录
rmdir --help #其他参数可以看帮助文档
```

### 2.3.4 rm 命令

```shell
#功能：删除文件或者目录 
rm -r  文件名或文件夹名 #删除并询问是否删除
rm -rf 名称		#强制删除，不询问
```

### 2.3.5 cp 命令

```shell
#功能:拷贝文件或目录
cp 123.txt demo/ #将文件123.txt拷贝到demo目录下
chmod 777 123.txt #改一下文件属性
cp -a 123.txt b.txt #拷贝123.txt并重命名b.txt , -a标识把文件的属性也一块复制

cp -r oldFolder /usr/local # 将oldFolder拷贝到新目录下，-r 表示递归
cp oldFile  /usr/local  #将文件拷贝到指定目录
```

### 2.3.6 mv 命令

```shell
#功能：移动文件或目录；或者改名
mv oldFolder newFolder	#修改文件夹名称
mv oldFile newFile  #修改文件的名称
mv oldFile  /usr/local  #将文件oldfile移动到/usr/local目录
mv oldFolder /usr/local #将目录移动到/usr/local目
```

### 2.3.7 diff 命令

```shell
#功能：对比文件差异
diff a.txt b.txt 
```

### 2.3.8 ssh 命令

```shell
#功能：远程安全登录方式
ssh 192.168.78.129 #从一台linux登录另一台机器，默认管理员账号
ssh user@hostname #指定登录用户
ssh -p 10022 user@hostname #ssh连接到其他端口
ssh root@192.168.78.129 ls -l #使用ssh在远程主机执行一条命令并显示到本地, 然后继续本地工作
```

### 2.3.9 exit 命令

```shell
#功能：退出登录命令
```

### 2.4.0 id 命令

```shell 
#功能：查看用户
[root@localhost ~]# id root
uid=0(root) gid=0(root) 组=0(root)
```

### 2.4.1 uname 命令

```shell
#功能：查询主机信息
[root@localhost ~]# uname -a
Linux localhost.localdomain 3.10.0-1160.53.1.el7.x86_64 #1 SMP Fri Jan 14 13:59:45 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

### 2.4.2 ping 命令

```shell
#功能：查看网络是否通
ping www.baidu.com
```

### 2.4.3 echo 命令

```shell
#功能：标准输出命令。 写shell脚本常用
echo "this is echo 命令"
```

### 2.4.4 clear 命令

```shell
#功能：清屏
clear #等同于快捷键 ctrl+l
```

### 2.4.5 who 命令

```shell
#功能：当前在本地系统上的所有用户的信息
whoami  #who am i 当前用户
who #当前有个用户登录
[root@localhost ~]# who
root     pts/0        2022-05-27 19:49 (192.168.78.1)
root     pts/1        2022-05-27 19:55 (192.168.78.1)
```

### 2.4.6 uptime 命令

```shell
#查询系统信息
[root@localhost ~]# uptime
19:57:12 up 18 min,  2 users,  load average: 0.00, 0.01, 0.05
#19:57:12 当前系统时间；up 18 min 从开启到现在运行时间；2 users 有两个用户登录；
#load average: 0.00, 0.01, 0.05 1分钟的负载，5分钟的负载，15分钟的负载
```

### 2.4.7 w 命令

```shell
#查询系统信息，可以看做uptime+who 的结合
[root@localhost ~]# w
 20:00:13 up 21 min,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.78.1     19:49    5.00s  0.02s  0.00s w
root     pts/1    192.168.78.1     19:55    4:42   0.01s  0.01s -bash
```

### 2.4.8 free 命令

### 2.4.9 wc 命令

### 2.5.0 grep 命令

### 2.5.1 find 命令

### 2.5.2 uniq 命令

### 2.5.3 sort 命令

### 2.5.4 df 命令

### 2.5.5 netstat 命令

### 2.5.6 hostname 命令

### 2.5.7 ps 命令

### 2.5.8 kill 命令

### 2.5.9 top 命令

### 2.6.0 du 命令

### 2.6.1 firewall-cmd 命令

### 2.6.2 echo 命令

### 2.6.4 cal 命令









## 2.3 输入输出重定向

## 2.4 编辑器vi的使用

## 2.5 用户管理与组管理

## 2.6 文件属性与权限操作

## 2.7 压缩与解压缩

# 3、linux必备核心使用命令

## 3.1 find命令-高级用法

## 3.2 Centos7的防火墙及 selinux介绍

## 3.3 telnet 与 scp命令用法

## 3.4 处理海量数据之 cut命令

## 3.5 处理海量数据之awk命令

## 3.6 处理海量数据之 sed命令

# 4、linux 常用服务软件的安装







