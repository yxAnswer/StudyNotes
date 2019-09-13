 # 一、Linux基础环境调试

这里采用的是centos7系统。

## 四项确认：

1、确认系统网络（因为我们得上网啊） 

ping一下就可以

2、确认yum可用 （因为我们得下东西啊）

`yum  list|grep gcc`     比如查个gcc ，随便试试就行

3、确认关闭iptables规则,或调试规则

`iptables -L` 看看有没有

`iptables -F` 有的话关闭

`iptables -t nat -L` 查看

`iptables -t nat -F` 关闭

4、确认停用selinux（学习期间关闭掉，有可能会影响一些规则）

`getenforce `  查看状态

`setenforce 0 `     关闭

## 两项安装：（基本库和工具）

`yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake`

`yum -y install wget httpd-tools vim`

## 初始化目录：（根据自己习惯）

`cd /opt;mkdir app download logs work backup `

app:  代码目录

download: 下载目录

logs:  自定义日志

work:  shell 等脚本

backup:  存放默认配置文件等的**备份**等