# Linux 实战技能100讲



## 1、linux背景知识

### linux的版本和分类

- Linux 有两种含义：
  - 一种Linus编写的开源操作系统的内核
  - 另一种是广义的操作系统

- Linux 版本
  - 内核版本
  - 发行版本

**内核版本**

- https://www.kernel.org/
- 内核版本分为三个部分（例如：5.0.15）
- 主版本号、次版本号、末版本号
- 次版本号是技术为开发板，偶数为稳定版

## 2、系统操作

### 1、万能的帮助命令

**（1）man 帮助**

man也是一条命令，分为9章，可以使用man命令来获取man的帮助

```shell
man ls  #查看ls相关
man man
man 7 man  #查看man第七章
```

**（2）help 帮助**

- shell(命令解释器）自带的命令成为内部命令，其他的是外部命令

- 内部命令使用help帮助

  -  `help cd`

- 外部命令使用help帮助

  - ls --help

- 如何区分内部命令外部命令；使用 `type`命令

 ```shell
[yangxu@localhost ~]$ type cd
cd 是 shell 内嵌
[yangxu@localhost ~]$ type ls
ls 是 `ls --color=auto' 的别名
 ```

**（3）info 帮助**

- info 帮助比help更详细，作为help的补充
  - 例如： `info ls`

### 2、文件管理

#### pwd 显示当前目录

pwd命令

- pwd 显示当前的目录名称

####  ls 目录下文件

ls命令

- **ls** 查看当前目录下的文件
  - ls [选项，...] 参数...
- 常见参数：
  - -l 	长格式显示文件
  - -a     显示隐藏文件
  - -r      逆序显示
  - -t      按照时间顺序显示
  - -R     递归显示

```shell
#参数可以合并比如：
ls -lrt  等于 ls -l -r -t    文件按照时间逆序
```

如果没有权限：  `su root` 切换root用户

#### cd 更改当前操作目录

cd 命令

- cd    更改当前的操作目录
  - cd /path/to/...绝对路径
  - cd ./path/to/...  相对路径，从当前目录开始，.和/ 可以省略
  - cd ../path/to/...  相对路径，从上一级开始

`使用.表示当前目录，..表示上级目录，/可以省略`

#### mkdir 建立目录

- mkdir 建立目录
- 常用参数：
  - -p 一次性建立多级目录
  - 例如： `mkdir  -p  a/b/c/d/e/f`

#### rmdir 删除空目录

- rmdir   删除空目录
- rm -r   删除非空目录

#### cp 复制文件

- cp     复制文件和目录

  - cp[选项]        文件目录  例如 cp  -r /a    /b   复制a目录到b目录
  - cp[选项]         文件...        路径  cp a.txt    b.txt   复制文件a.txt 为b.txt

  常用参数：

- -r     复制目录

- -p     保留用户、权限、时间等文件属性

- -a     等同于-dpR ，更极端保留的更全

- -v     显示复制的过程

#### mv 移动文件

- mv    1、 移动文件或文件夹； 2、重命名

  - mv [选项]   源文件    目标文件 
  - mv [选项]   源文件    目录

  ```shell
  mv /a /b   #将a目录重命名b 目录，如果有了b目录,就是移动
  mv /a /temp   #将a目录移动到/temp目录下 
  ```

#### rm 删除文件

- rm 删除文件
- 常用参数：
  - -r   删除目录（包括目录下的所有文件）
  - -f   删除文件不进行提示
- 注意：rm 命令可以删除多个目录，需谨慎使用

#### 通配符

## 3、系统管理

## 4、shell

## 5、文本操作

## 6、服务篇 

## 7、实战

常见目录介绍

- / 根目录
- /root  root用户的 home目录
- /home/username 普通用户的home目录
- /etc 配置文件目录
- /bin 命令目录
- /sbin管理命令目录
- /usr/bin  /usr/sbin 系统预装的其他命令





