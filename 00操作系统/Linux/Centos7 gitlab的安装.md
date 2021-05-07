# Centos7  gitlab的安装

## 1、切换国内的源

备份/etc/yum.repos.d/CentOS-Base.repo

```shell
#先备份源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
#下载centos7对应的repo文件，放入/etc/yum.repos.d  使用的是网易163的源
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
mv CentOS7-Base-163.repo CentOS-Base.repo
#生成缓存
yum clean all
yum makecache
```

更新软件：

```shell
yum upgrade 

```

如果出现了错误，比如，出现包冲突、重复包，损坏包等问题：

```shell
yum install yum-utils
yum-complete-transaction --cleanup-only
#清除可能存在的重复包
package-cleanup --dupes
#清除可能存在的损坏包
package-cleanup --problems
#清除重复包的老版本：
package-cleanup --cleandupes
```

### 虚拟机使用物理主机的代理，科学上网（可选）

练习的时候，如果使用的虚拟机，那么主机有代理软件，那么

- 虚拟机选择网络使用桥接模式（桥接模式就是：虚拟机和主机在同一个网段，和其他内网的电脑没啥区别，会被分配ip，但是有ip冲突的风险。）

- 物理主机的代理软件比如ssr,v2rayN等设置允许局域网访问

- 虚拟机linux设置代理

- ```shell
  export http_proxy=http://xxxxxx:1080 # 设置物理主机的ip和代理的端口号
  export http_proxy=http://xxxxxx:1080
  ```

- 取消代理：

- ```shell
  unset http_proxy
  unset https_proxy
  ```

## 2、配置防火墙

```shell
yum install firewalld systemd -y #安装防火墙
systemctl start  firewalld.service #开启防火墙

yum install -y curl policycoreutils-python openssh-server perl #安装ssh协议
#如果报错，rpm --rebuilddb && yum install -y curl policycoreutils-python openssh-server perl 
#重建数据库

systemctl enable sshd #设置ssh服务开机启动
systemctl start sshd #启动ssh服务
firewall-cmd --permanent --add-service=http #添加http服务都firewalld
firewall-cmd --permanent --add-service=https #添加HTTPS服务到firewalld
systemctl reload firewalld #重启防火墙

```

## 3、安装prostfix发送email服务

```shell
yum install postfix #安装Postfix以发送通知邮件
systemctl enable postfix #将postfix服务设置成开机自启动
systemctl start postfix #启动postfix

#最好装一下vim,后面会用到
yum install vim -y
```



## 4、添加gitlab仓库，并安装

官网地址：[https://about.gitlab.com/install/?version=ce#centos-7](https://about.gitlab.com/install/?version=ce#centos-7)

选择的是免费的社区版本：gitlab-ce

```shell
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

yum install -y gitlab-ce
```



## 5、启动gitlab

```shell
gitlab-ctl reconfigure # 自动配置(可选)，当然也可以全部修改完以后再执行，执行完就可以访问网站了

vim /etc/gitlab/gitlab.rb # 修改
修改external_url为gitlab机子的ip+要使用的端口 如：http://192.168.1.141:8888 
修改nginx['listen_port'] = 8888 
修改时区gitlab_rails['time_zone'] = 'Asia/Shanghai'
重新配置gitlab并重启 
gitlab-ctl reconfigure 
gitlab-ctl restart
```

vim 小技巧搜索：

 命令行模式输入/要搜索的内容比如： /listen_port ，就会搜索到，然后按n 跳转下一个匹配，N回到上一个匹配



## 6、配置防火墙

修改完配置文件，会出现无法访问的情况，需要修改一下防火墙，开放设置的端口

```shell
#开放8888端口
firewall-cmd --zone=public --add-port=8888/tcp --permanent   

#重启防火墙
firewall-cmd --reload
#查看端口号是否开启
firewall-cmd --query-port=8888/tcp

```

## 7、访问gitlab网站

- http://xxxxxx:8888   这个就是我们gitlab所在的服务器ip+我们配置的端口，就可以访问
- 重新设置root 密码
- 登录，尽情使用把

## 8、gitlab配置邮件服务

邮件服务的作用：有合并请求时，邮件通知 账号注册时，邮件验证 修改密码时，通过邮件修改  。

- 尽量不要用163邮箱，因为发送几次就不能发送了。可以用qq邮箱。

- 打开QQ邮箱——>设置——>账户——>smtp——>密保验证——>验证成功返回一串字符串，保存下来。

- 修改gitlab配置文件

  ```
  vim /etc/gitlab/gitlab.rb # 修改
  按/后输入smtp_enable，找到下面这一串文本，进行修改
  gitlab_rails['smtp_enable'] = true
  gitlab_rails['smtp_address'] = "smtp.qq.com"
  gitlab_rails['smtp_port'] = 465
  gitlab_rails['smtp_user_name'] = "xxxxxxxx@qq.com"
  gitlab_rails['smtp_password'] = "开通smtp时返回的字符"
  gitlab_rails['smtp_domain'] = "qq.com"
  gitlab_rails['smtp_authentication'] = "login"
  gitlab_rails['smtp_enable_starttls_auto'] = true
  gitlab_rails['smtp_tls'] = true
  user['git_user_email'] = "xxxxxx@qq.com"
  gitlab_rails['gitlab_email_from'] = 'xxxxxxx@qq.com
  
  # 在这里配置好自己的邮箱即可，然后保存，重新配置
  gitlab-ctl reconfigure
  
  ```

- 测试邮件服务是否正常

```shell
gitlab-rails console #进入控制台 
#然后，输入
Notify.test_email('接收方邮件地址','邮件标题','邮件内容').deliver_now
# 按回车，测试发送。
```

