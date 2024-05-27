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

- 关闭防火墙：`systemctl stop firewalld.service   `

- 设置selinux 为disable  ：` vi /etc/selinux/config`

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

- 更新xfsprogs 文件系统工具 ：`yum -y update xfsprogs`

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
- 本地镜像的删除：`docker rmi centos:7` 或  `docker rmi -f centos:7`  ，等同于`docker image rm centos:7`

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
  - **-v** :映射宿主机路径到容器内部，用于文件持久化。`-v /data:/home  -v  /usr/local/app1:/usr/local` 可以映射多个
  - **-p** : 映射容器端口和宿主端口。`-p 8080:80 -p 6379:6379` 可以指定多个
- 查看本地所有的容器： `docker ps -a`
- 查看本地正在运行的容器：`docker ps`
- 停止容器：`docker stop CONTAINER_ID / CONTAINER_NAME`
- 一次性停止所有容器：`docker stop $(docker ps -a -q)`
- 启动容器：`docker start CONTAINER_ID / CONTAINER_NAME`
- 重启容器：`docker restart CONTAINER_ID / CONTAINER_NAME`
- 删除容器：`docker rm CONTAINER_ID / CONTAINER_NAME`
- 强制删除容器：`docker rm -f CONTAINER_ID / CONTAINER_NAME`
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
docker rm -f mycentos

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
  - 如果一个创建好的容器，想查看哪些路径是映射的，使用`docker inspect mycentos`查看容器信息，查看Mounts部分，Source为宿主机路径，Destination为容器内路径。

# 3. docker 自定义镜像

docker自定义镜像大概有几种方式：

- 基于Docker Commit制作镜像，适用于快速修改并创建个镜像，但不够灵活；
- 基于dockerfifile制作镜像，Dockerfifile方式为主流的制作镜像方式（推荐）。提供了更结构化、可管理可重复的方式构建镜像。
- 使用Docker Compose 允许定义和运行多个容器，一般不用来自定义镜像，但是如果指定了dockerfile运行的时候也会构建镜像

## 3.1 Commit 构建自定义镜像

```shell
#创建一个测试容器
docker run -itd --name=mycentos  centos:7
#进入容器
docker exec -it mycentos /bin/bash 
#将容器进行定制，比如安装软件，或者其他操作
cd /home
mkdir  testFolder 
#使用commit命令自定当前容器为新的本地镜像 docker commit  容器ID或名  新的镜像名称
docker commit  -a "标注作者"  -m "输入一些容器描述" mycentos  test-image:7
#查看镜像详细信息
docker inspect test-image:7
#然后删除现有容器，用新创建的镜像重新运行一个容器，会发现之前所做的操作都在
docker rm -f mycentos 
docker run -it test-image:7 /bin/bash 


```



## 3.2 DockerFile 介绍及常用命令

DockerFile常用指令：

- FROM  : 指定构建镜像的基础镜像

- MAINTAINER：已被弃用。用于设置镜像作者

- COPY ：复制文件或目录进入镜像（只能用相对路径，不能用绝对路径）

- ADD：复制文件或目录进入镜像（假如文件是.tar.gz文件会自动解压，还可以从URL获取文件）

- WORKDIR：指定Dockerfile中后续指令的工作目录，也就是容器中的工作目录，假如路径不存在会创建路径

- ENV：以 `key=value` 的形式设置环境变量。这些变量在容器运行时持续存在。

- EXPOSE：通知 Docker 容器会在运行时监听指定的网络端口。它并不会真正发布这些端口。

- RUN ：在构建镜像的时候执行命令。可以使用 shell 形式或 exec 形式。

-  ENTRYPOINT ：在容器启动的时候执行，作用于容器层，dockerfifile里有多条时只允许执行最后一条。它允许你设置默认的可执行文件及其参数。

- CMD ：在容器启动的时候执行，作用于容器层，dockerfifile里有多条时只允许执行最后一条。

  - 用于为 ENTRYPOINT 指令提供默认参数，或者指定容器启动时要执行的命令。
  - 比如： ENTRYPOINT 执行ps  ,CMD 执行-ef ,那么容器启动的时候就是直接回运行`ps -ef`
  - 假如说我们启动容器的时候，比如：docker run 容器ID   `aux` ，那么最终执行的命令是`ps -aux` ，也就是说CMD指定的命令被覆盖了
  - 总结：docker run 启动容器时后面如果跟上其他命令，会覆盖Dockerfile中设置的CMD的命令；

- 命令格式：

  - **shell命令格式**，例如：RUN yum install -y net-tools
  - **exec命令格式**，例如：RUN [ "yum","install" ,"-y" ,"net-tools"]

  

**测试一个例子：**

DockerFile创建：

```shell
# this is a dockerfile 
FROM centos:7 
MAINTAINER zhhangsan 123456@qq.com 
RUN echo "正在构建镜像！！！" 
WORKDIR /home/demo 
COPY 123.txt /home/demo 
RUN yum install -y net-tools
```

构建镜像：


```shell
#构建一个新的镜像
docker build -t  demo:1  .   #注意这个是命令行当前目录下有这个Dockerfile ，这里有个点.,表示当前目录
#或者
docker build -t demo:1  DockerFile所在目录  
#有一个问题，就是我测试时用的macos，当我的Dockerfile命名为DockerFile 就不行，命名Dockerfile或dockerfile都可以

```

docker build 在构建过程中是一层一层的构建的，比如根据dockerfile内容，从最开始From的初始镜像，执行不同的命令分别构建不同的镜像层，一层层叠加成最后生成的镜像。

- RUN 、ENTRYPOINT、CMD都能执行命令，他们的区别是什么？

  - RUN 在构建镜像的时候执行，作用于镜像层面

  - ENTRYPOINT   在容器启动的时候执行，作用于容器层，dockerfile里有多条时只允许执行最后一条

  - CMD  在容器启动的时候执行，作用于容器层，dockerfile里有多条时只允许执行最后一条。容器启动后执行默认的命令或者参数，允许被修改

  - ```shell
    #第一个 
    FROM centos:7 
    RUN echo "images building!" 
    CMD ["echo","container","starting..."] 
    ENTRYPOINT ["echo","container","starting ！！！"] 
    
    #第二个 
    FROM centos:7 
    RUN echo "images building1!" 
    RUN echo "images building2!" 
    CMD ["echo","containe1r","starting..."] 
    CMD ["echo","container2","starting..."] 
    ENTRYPOINT ["echo","container2","starting ！！！"] 
    ENTRYPOINT ["echo","container2","starting ！！！"] 
    
    #第三个 
    FROM centos:7 
    CMD ["-ef"] 
    ENTRYPOINT ["ps"]
    ```

  - 比如上面有三个不同的dockerfile会产生不同的结果：

    - 第一个：三个命令都会输出，说明 这三个指定都可以执行命令，RUN构建镜像时执行，CMD和ENTRYPOINT 启动容器时执行。
    - 第二个：三个命令都会输出，但是CMD和ENTRYPOINT 只会输出最后一条；说明这两个指令只允许执行最后一条命令，RUN 可以重复执行
    - 第三个：启动容器时会执行`ps -ef` ，说明先执行ENTRYPOINT 再执行CMD，也就是说CMD可以为ENTRYPOINT设置默认参数； 当我们启动容器时后面接上其他命令，会覆盖CMD，比如 docker run mycentos  -aux  ,会执行ps -aux
    
  - **注意： 当dockerfile里面的命令采用的是exec命令格式 时， ENTRYPOINT和CMD 的命令是可以结合起来的，比如ps -ef ; 但是当采用的是shell命令格式的时候，就是一个单独的命令各自执行各自的命令，CMD 的命令并不会被完全覆盖，而是会拼接到CMD 指定的shell命令后面。**





# 4. docker 网络模式与特权

## 4.1 docker容器的网络模式介绍

简介：介绍Docker网络模式

- 默认的三种网络模式：
  - **bridge**：桥接模式
  - **host**：主机模式
  - **none**：无网络模式
- 查看网络模式：
  - docker network ls

### 4.1.1 bridge：桥接模式

简介：介绍Docker桥接网络模式

- 桥接模式是docker 的默认网络设置，当Docker服务启动时，会在linux主机上创建一个名为docker0的虚拟网桥，并选择一个和宿主机不同的IP地址和子网分配给docker0网桥
- 网络访问：公网----> 宿主机网卡------通过NAT----->  docker0网卡----> 各个容器的网卡
- centos安装网络工具：
  - yum  -y install net-tools
  - yum install -y  bridge-utils
- brctl show  查看桥接情况
- route -n 查看路由表

### 4.1.2 host: 主机模式

- host 模式：该模式下容器是不会拥有自己的ip地址，而是使用宿主机的ip地址和端口。
- 网络访问：公网----> 宿主机网卡/各个容器的网卡(容器与宿主机网格栈共享，不单独分配ip)
- 指定网路模式： --net=host ，比如： `docker  run -itd  --net=host   --name=mycentos    centos:7  `

### 4.1.3 none: 无网络模式

- none模式：关闭模式
- 无法连外网，比较少用，一般测试用

## 4.2 docker容器间基于Link实现单向通信

- 单向通信，指的是两个容器之间通过容器名可以进行网络通信，A能解析host得到B容器ip，然后访问容器B； B容器不能解析host通过容器名访问容器A，这就是所谓的单向通信； 
- 为什么要通过容器名访问？例如：当一个数据库容器或者其他的公共服务容器，ip地址改了以后，那么所有已经运行的访问此ip的容器是不是需要重新修改配置。但是如果是通过解析容器名直接访问，就不用修改了，也就是解析了host。
- 举例： docker run -itd --name tomcat1 --link mydb tomcat:tag   ，启动一个tomcat1容器，并设置单向通信到mydb这个容器（一定存在）
- `--link 容器名`： 配置容器间的单向通信

## 4.3 docker容器间基于bridge实现双向通信 

- 查看网络模式  `docker network ls `
- 如果是先双向通过容器名通信呢：  通过建立一个新的网桥，所有容器都加入网桥即可
- 创建一个新的网桥：**docker network create -d bridge my_bridge**
- 启动第一个容器：**docker run -itd --name tomcat centos:7**
- 启动第二个容器：**docker run -itd --name redis centos:7**
- 把第一个容器加入网桥：**docker network connect my_bridge tomcat**
- 把第二个容器加入网桥：**docker network connect my_bridge redis**
- 最后分别进入俩个容器中进行验证

## 4.4 docker容器的特权模式介绍（一般不用）

Docker 的特权模式（Privileged Mode）是一种容器运行模式，它提供了对宿主机系统的更高权限访问。在特权模式下，容器内的进程拥有与宿主机上的进程相同的权限，这使得容器内的操作更接近于宿主机上的操作。以下是 Docker 的特权模式的一些关键特点和使用情境：

1. **宿主机权限：** 在特权模式下，容器内的进程拥有宿主机上的 root 权限。这意味着容器内的操作可以对宿主机系统的资源进行更广泛的访问，包括访问设备、更改网络配置、修改内核参数等。
2. **容器内的 / 目录映射到宿主机的 / 目录：** 特权模式允许容器内的 / 目录映射到宿主机的 / 目录，使得容器内的操作能够对宿主机的根文件系统产生影响。
3. **设备访问：** 特权模式允许容器访问宿主机上的设备，包括 /dev 目录下的设备。这对于一些需要直接操作设备的应用程序或服务是有用的。
4. **CAP_SYS_ADMIN 权限：** 特权模式赋予容器内的进程 CAP_SYS_ADMIN 权限，这是 Linux 内核中的一个权限标记，允许进程执行一些高级系统管理任务。

使用特权模式时需要谨慎，因为容器内的操作可能对宿主机系统产生较大影响，并增加了潜在的安全风险。在一般情况下，应尽量避免使用特权模式，除非确实需要进行高级的系统操作。

使用特权模式的方式是在运行容器时使用 `--privileged` 参数，例如：

```
docker run -itd --privileged=true --name mycentos1 centos:7 /bin/bash
```

需要注意的是，使用特权模式是一种潜在的安全风险，只应在了解其影响并确信有必要时使用。



## 4.5 docker容器之Volumn数据共享

- Docker 的 Volume 是一种用于在容器和宿主机之间共享、持久化存储数据的机制。Volumes 提供了一种方便的方式来处理容器中的数据，并且与容器的生命周期无关，这意味着即使容器被删除，Volume 中的数据仍然保留

- 绑定挂载:  `docker run -v /宿主机路径:/容器内路径 myimage` ,指定宿主机的路径和容器内的路径绑定，这样实现数据持久化

- 命名卷： 

  - `docker volume create mydata` ,创建一个名为mydata 的卷

  - `docker volume  inspect mydata`: 可以查看这个卷的信息，比如：Mountpoint 就是宿主机的实际路径

  - ```json
      [
          {
              "CreatedAt": "2024-01-22T07:18:37Z",
              "Driver": "local",
              "Labels": null,
              "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
              "Name": "mydata",
              "Options": null,
              "Scope": "local"
          }
      ]
      ```

  - 使用命名卷：`docker run -v mydata:/app/data myimage`      将命名卷挂载到容器内的指定路径，这样的命名卷在宿主机上由 Docker 管理，提供了持久性、共享和数据管理的优势

- 常用命令

  - ```shell
    #创建命名卷
    docker volume create mydata   
    #查看所有卷
    docker volume ls
    #查看卷的详细信息
    docker volume inspect mydata
    #删除卷
    docker volume rm mydata
    #挂载命名卷到容器
    docker run -v mydata:/app/data myimage
    #挂载宿主机目录到容器
    docker run -v /host/path:/container/path myimage
    
    
    ```

- ` --volumes-from ` : 用于指定容器应该挂载自另一个容器的所有挂载点（Volumes）。这个选项允许一个容器共享它的 Volume（卷）给另一个容器，使得这两个容器可以共享数据。例如：

- ```shell
  #1、假设app_data 有一个命名卷mydata ,或者是挂载绑定映射的宿主机路径也可以。
  docker run -d --name app_data -v mydata:/app/data myimage_data
  #2、新建容器，共享另一个容器的卷
  docker run -d --name app_web  --volumes-from app_data myimage_web
  # 这样，app_web 就能够访问 app_data 容器中的 mydata 卷。这在一些情况下很有用，例如将数据容器与应用容器分离，实现数据持久化和应用容器的分离
  
  ```

  

# 5. docker Compose的使用

## 5.1 docker compose介绍安装

简介：Docker Compose 是 Docker 官方提供的一个用于定义和运行多容器 Docker 应用的工具。它允许你使用一个单独的文件（`docker-compose.yml`）来配置应用的服务、网络和卷，然后使用一条简单的命令 (`docker-compose up`) 就能启动整个应用。

**安装docker  compose**

建议直接参考官网安装：https://docs.docker.com/compose/install/#install-compose

默认情况下，windows 和 mac 下的 docker 已经自带了 docker-compose 工具，可以使用 `docker-compose -v` 命令查看。

linux系统需要手动安装：

- 通过官网下载安装，参考[官方文档](https://docs.docker.com/compose/install/#install-compose)
- 通过pip安装 ,自行百度

## 5.2 docker compose 常用命令

**操作docker-compose一定要在配置文件docker-compose.yml文件路径下操作，也可以手动指定路径：**

```shell
docker-compose -f /path/to/docker-compose.yml up
```

 **应用容器管理：**

```shell
docker-compose up    						#启用应用  （会先构建镜像然后依次启动容器）
docker-compose up -d 						#后台启动应用，不会阻塞终端
dokcer-compose ps    						#查看运行中的服务

docker-compose stop							#停止应用 (不占用内存和cpu)
docker-compose start						#启动应用
docker-compose down  						#停止并删除容器
docker-compose down --volumes  	#停止并删除容器，并删除卷volumes

docker-compose build					#构建镜像：如果docker-compose.yml配置了需要构建的容器，Dockerfile等，会构建镜像，但不启动容器
```

**配置信息查看：**

```shell
docker-compose config			#查看compose文件配置
docker-compose logs				#查看compose日志
docker-compose version		#查看compose版本

```

**服务控制：**

```shell
docker-compose pause			#暂停容器(容器冻结，不在运行，内存cpu处于冻结状态)
docker-compose unpause		#恢复容器
```

**执行命令：**

```shell
docker-compose exec [service] [command]  #使用exec进入容器
docker-compose exec redis bash
```

## 5.3 docker-compose.yml 文件介绍

**以MongoDB、Elasticsearch、GrayLog为例：**

```yaml
version: '3'    #指定 Docker Compose 文件格式的版本
services:  			# 定义应用中的服务（容器）
	#MongoDB 服务配置
  mongodb:			
    image: mongo:5.0.13		#使用 MongoDB 5.0.13 镜像
    volumes:
      - mongo_data:/data/db   #挂载名为mongo_data的卷到容器的 /data/db目录
      
	#Elasticsearch 服务配置
  elasticsearch:		
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.10.2
    volumes:
      - es_data:/usr/share/elasticsearch/data
    environment:		#设置环境变量
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"		#设置 Elasticsearch 的 Java 虚拟机参数
    ulimits:				#容器的资源限制
      memlock:			#内存锁定
        soft: -1		#软限制：-1表示没有限制（软限制超过会警告，不会阻止进程运行）
        hard: -1		#硬限制：-1表示没有限制
    mem_limit: 1g		#限制容器使用的内存为 1GB
    
	#Graylog 服务配置
  graylog:				  
    image: graylog/graylog:5.0		#使用Graylog 5.0 镜像
    volumes:
      - graylog_data:/usr/share/graylog/data
    environment:
      # CHANGE ME (must be at least 16 characters)!
      - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
      # Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_HTTP_EXTERNAL_URI=http://127.0.0.1:9000/
    entrypoint: /usr/bin/tini -- wait-for-it elasticsearch:9200 -- /docker-entrypoint.sh
    links:
      - mongodb:mongo
      - elasticsearch
    restart: always  		#总是在容器退出时重启
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 1514:1514
      # Syslog UDP
      - 1514:1514/udp
      # GELF TCP
      - 12201:12201
      # GELF UDP
      - 12201:12201/udp

# 命名卷，用于持久化数据
volumes:
  mongo_data:
    driver: local
  es_data:
    driver: local
  graylog_data:
    driver: local
```

**常用属性说明：**

- `version`: Docker Compose 文件的版本。不同版本可能支持不同的特性和语法。常见的版本有 "2", "3", "3.1", 等等。每个版本都有不同的语法和功能
- `services`: 这是定义各个服务的地方。一个服务通常对应一个容器，定义了容器的各种配置。
  - `image`: 定义服务使用的镜像及版本。
  - `build`: 用于指定 Dockerfile 的路径，从而构建自定义镜像。
  - `volumes`: 定义数据卷挂载。
  - `environment`: 设置环境变量。
  - `ulimits`: 设置资源限制。在这个例子中，`memlock` 限制设置了容器可以锁定的内存量，soft 和 hard 分别表示软限制和硬限制。
  - `mem_limit`: 设置容器的总体内存限制。它指定容器可以使用的最大内存量。
  - `entrypoint`: 指定容器的入口点。入口点是容器启动时执行的命令或脚本。
  - `links`: 定义服务之间的链接。在这个例子中，`graylog` 服务链接到 `mongodb` 和 `elasticsearch`，这样 `graylog` 可以通过这些链接访问其他服务。
  - `restart`: 设置容器的重启策略。在这个例子中，`always` 表示容器总是在停止时重新启动。
  - `depends_on`: 定义服务之间的依赖关系。在这个例子中，`graylog` 依赖于 `mongodb` 和 `elasticsearch`，这意味着在启动 `graylog` 之前，`mongodb` 和 `elasticsearch` 必须先启动。
  - `ports`: 定义端口映射。将容器内的端口映射到宿主机上的端口，使外部可以访问容器内的服务。
- `networks`: 定义服务使用的网络。网络配置允许服务之间进行通信，也可以配置其他网络属性。

参考示例结构：

```yaml
version: '3.1'

services:
  my_service:
    image: my_image:latest
    build: ./path/to/Dockerfile
    volumes:
      - /host/path:/container/path
    environment:
      - ENV_VAR1=value1
      - ENV_VAR2=value2
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
    entrypoint:
      - "/bin/sh"
      - "-c"
      - "echo Hello, world!"
    links:
      - mongodb:mongo
      - elasticsearch
    restart: always
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      - "8080:80"
      - "8443:443"

networks:
  my_network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: my_bridge
      com.docker.network.bridge.enable_icc: "false"
    ipam:
      config:
        - subnet: "172.20.0.0/16"
          gateway: "172.20.0.1"
        - subnet: "192.168.0.0/24"
          gateway: "192.168.0.1"

```



# 6. docker  镜像仓库 

## 6.1 企业harbor仓库搭建

## 6.2 企业harbor仓库配置与使用

## 6.3 本地镜像容器的载入与载出







