# Linux 入门到实战

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
cd /etc/sysconfig/network-scripts  #到此目录


```



## 1.3 vmware虚拟机的备份与快照恢复

## 1.4 虚拟机centos7与外部物理机的时间同步

# 2、linux常用操作

## 2.1 linux目录分类介绍 

## 2.2 工作中常用基础命令

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







