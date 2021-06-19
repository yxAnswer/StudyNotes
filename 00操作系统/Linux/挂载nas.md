# 挂载Windows 共享

```shell
yum install -y cifs-utils #安装cifs-utils 挂载windows共享盘
```



```shell
#挂载windows共享盘
mount -t cifs //192.168.1.195/nas /mnt/linux_nfs/ -o username=Administrator,password=BimEx_2021_S9,iocharset=utf8,dir_mode=0777,file_mode=0777

```

```shell
#卸载
umount /mnt/linux_nfs
```

```java
//判断当前系统是windows还是linux，然后从数据库读取路径
String os = System.getProperty("os.name");
```

