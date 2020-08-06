# 选择镜像

```shell
docker pull ubuntu
```

# 创建容器

```shell
docker create -ti --privileged --name work -p 10000:80 -p 10001:8888 -v D:\DockerWorkSpace\dockerwork:/root/work -w /root ubuntu bash -l
```

- -ti 交互模式
- --privileged 特权模式。（以便后续使用GDB等工具）
- --name 容器名称
- -p 10000:80 将容器内的80端口映射到宿主机 10000端口。这样可以在容器内启动webserver
- -p 10001:8888 将容器内8888映射主机10001，方便使用jupyter notebook 在主机访问
- -v  本机路径:容器路径  将宿主目录 绑定到容器目录， 可以在本机写代码，容器内运行，同时方便容器内处理文件保存到主机，不丢失
- -w 容器的工作目录
- ubuntu 容器基础镜像名称
- bash -l 启动容器shell进程

# 启动容器

```shell
docker start -ai work --启动名为work 的容器
```

# 安装基础软件

```shell
apt-get update
apt install curl
apt install python3.8 python3.8-distutils
--安装 pip包管理器
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.8 get-pip.py
pip3 -V  --测试是否安装成功

--安装附加工具，挺好用的
pip3 install ipython ipdb jupyter
```

# notebook学习环境

```shell
$ jupyter notebook password  --设置个密码
Enter password:
Verify password:
$ jupyter notebook --ip=* --allow-root
--这就启动了服务，可以在宿主电脑访问 http://localhost:10001 访问
--默认是8888  上面把主机10001 映射到容器 8888端口

```



常用命令

```shell
$ docker images --查看所有镜像
$ docker attach  容器名
```

