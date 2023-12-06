# Docker小结（一）

[toc]

# 1. docker 的安装

## 1.1 什么是docker 

docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任 何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口； 使用go语言编写，在LCX（linux容器）基础上进行的封装 

- 可以快速部署启动应用 
- 实现虚拟化，完整资源隔离 
- 一次编写，四处运行（有一定的限制，比如Docker是基于Linux 64bit的，无法在32bit的 linux/Windows/unix环境下使用） 

## 1.2 linux centos7安装docker 

docker官网：https://docs.docker.com/engine/install/centos/

- Docker 版本：

  - docker EE 企业版本

  - docker CE 社区版本

- 关闭防火墙：systemctl stop fifirewalld.service vi /etc/selinux/confifig

- 安装wget命令：

- 下载阿里云docker社区版 yum源

- ```shell
  [root@localhost ~]# cd /etc/yum.repos.d/ 
  [root@localhost yum.repos.d]# 
  [root@localhost yum.repos.d]# wget http://mirrors.aliyun.com/docker- ce/linux/centos/docker-ce.repo
  ```

- 查看docker安装包：`yum list | grep docker`

- 安装Docker Ce 社区版本：`yum install -y docker-ce.x86_64`

- 设置开机启动：`systemctl enable docker`

- 更新xfsprogs：`yum -y update xfsprogs`

- 启动docker：`systemctl start docker`

- 查看版本：`docker version`

- 查看详细信息：`docker info`

# 2. docker 的基本使用

## 2.1 docker 镜像使用

- 查看本地镜像：`docker images`
- 搜索镜像：`docker search centos`
- 搜索镜像并过滤是官方的：`docker search --filter "is-official=true" centos`
- 搜索镜像并过滤大于多少颗星星的：`docker search --filter stars=10 centos`
- 下载centos7镜像：`docker pull centos:7`
- 修改本地镜像名字（小写）：`docker tag centos:7 mycentos:7`
- 本地镜像的删除：`docker rmi centos:7`

## 2.2 配置阿里镜像加速

阿里云地址：https://cr.console.aliyun.com/cn-shenzhen/instances/mirrors

需要登录阿里云账号，每个账号会有一个域名地址。详细各个系统配置可以参考阿里云网站。

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://t8p44qc9.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2.3 docker  容器使用

- 构建容器： `docker run -itd --name=mycentos centos:7`
  - **-i** ：表示以交互模式运行容器（让容器的标准输入保持打开）
  - **-d**：表示后台运行容器，并返回容器ID
  - **-t**：为容器重新分配一个伪输入终端
  - **--name**：为容器指定名称
- 查看本地所有的容器： `docker ps -a`
- 查看本地正在运行的容器：`docker ps`
- 停止容器：`docker stop CONTAINER_ID / CONTAINER_NAME`
- 一次性停止所有容器：`docker stop $(docker ps -a -q)`
- 启动容器：`docker start CONTAINER_ID / CONTAINER_NAME`
- 重启容器：`docker restart CONTAINER_ID / CONTAINER_NAME`
- 删除容器：`docker rm CONTAINER_ID / CONTAINER_NAME`
- 强制删除容器：`docker rmi -f CONTAINER_ID / CONTAINER_NAME`
- 查看容器详细信息：`docker inspect CONTAINER_ID / CONTAINER_NAME`
- 进入容器：`docker exec -it 0ad5d7b2c3a4 /bin/bash`

```shell
#构建容器：
docker run -itd --name=mycentos centos:7
#查看本地所有的容器： 
docker ps -a 
#查看本地正在运行的容器：
docker ps

nick@nickdeMacBook-Pro ~ % docker ps 
CONTAINER ID   IMAGE      COMMAND       CREATED          STATUS          PORTS     NAMES
c20200452a9f   centos:7   "/bin/bash"   50 seconds ago   Up 49 seconds             mycentos

#启动容器：
docker start mycentos
#查看容器详细信息：
docker inspect mycentos
#进入容器：
docker exec -it mycentos /bin/bash
#重启容器：
docker restart mycentos
#停止容器：
docker stop mycentos
#一次性停止所有容器：
docker stop $(docker ps -a -q)
#删除容器：
docker rm mycentos
#强制删除容器：
docker rmi -f mycentos

```

## 2.4 docker 容器文件复制与挂载

例如：容器名字为mycentos 

- 从宿主机复制到容器：**docker cp    宿主机本地路径    容器名字/ID:容器路径**
  - `docker cp   /root/123.txt     mycentos:/home/`

- 从容器复制到宿主机：**docker cp   容器名字/ID:容器路径    宿主机本地路径**
  - `docker cp  mycentos:/home/456.txt   /root`

- 宿主机文件夹挂载到容器里：**docker run -itd    -v  宿主机路径:容器路径 镜像ID**
  - `docker  run  -itd   -v /root/data/:/home  centos:7`
  - 运行容器的时候指定本地文件的映射，用于容器数据持久化

# 3. docker 自定义镜像

# 4. docker 网络模式与特权

# 5. docker Compose的使用

# 6. docker  镜像仓库 





