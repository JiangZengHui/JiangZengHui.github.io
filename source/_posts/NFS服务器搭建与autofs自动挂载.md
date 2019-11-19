---
title: NFS服务器搭建与autofs自动挂载
top: false
cover: false
toc: true
mathjax: true
date: 2019-10-26 20:04:12
password:
summary: NFS服务器搭建与autofs自动挂载
tags: Linux网络运维
categories: Linux
---


#### 一、NFS和autofs简介

NFS（Network File System），网络文件系统是 Linux 系统支持的一种网络服务，通过 NFS ，网络中的计算机可以发布共享信息，让远程客户像使用本地文件一样访问该共享资源，若想使用远程计算机上的文件，只要用 mount 命令将远程的目录挂载在本地文件系统下即可。
Autofs：mount是用来挂载文件系统的，可以在系统启动的时候挂载也可以在系统启动后挂载。对于本地固定设备，如硬盘可以使用mount挂载；而光盘、软盘、NFS、SMB等文件系统具有动态性，即需要的时候才有必要挂载。autofs服务就提供这种功能，好像windows中的光驱自动打开功能，能够及时挂载动态加载的文件系统，免去我们手动挂载的麻烦。

#### 二、服务端NFS安装

```
yum install nfs-utils -y   #安装nfs
systemctl start nfs   #开启nfs服务
systemctl enable nfs   #开机自启动
```

#### 三、配置NFS服务

```
mkdir /westos
echo 'hello,world' > /westos/hello  
# 建立目录 /westos, 在其中创建测试文件hello
```

> NFS 的主配置文件为 /etc/exports，该文件中可以设置 NFS
> 的共享目录、访问权限和允许访问的主机等参数；默认情况下是空文件，不配置任何共享目录

#### 四、服务端测试

showmount命令查看指定服务器的nfs共享文件信息，常用选项 -e：显示指定服务器输出的共享目录

```
showmount -e 192.168.1.152
Export list for 192.168.1.152:
/westos *  
```

#### 五、挂载共享目录

```
mount 服务器名或IP地址：共享目录 本地挂载目录
# 将共享目录挂载到本地/mnt     ls /mnt/
mount 192.168.1.152:/westos /mnt/ 
hello   #有我们的测试文件，说明目录共享成功 
vim /etc/fstab 
# 让共享目录自动挂载
192.168.1.152:/westos /mnt/ nfs defaults 0 0 
```

#### 六、autofs的安装

```
yum install autofs -y
systemctl start autofs.service
```

#### 七、特殊映射/net

> autofs服务开启之后，将自动生成/net目录，默认将共享目录挂载在该目录中，只要使用 cd 命令指定 NFS
> 服务器的IP地址，就可以直接挂载使用远程主机上的 NFS 共享

```
# 使用cd命令时就会自动挂载共享目录
cd /net/192.168.1.152/westos              
ls
hello
```

#### 八、自定义卸载时间

```
# 等待时间配置文件
vim /etc/sysconfig/autofs      
# 切出共享目录路径，5秒后就自动卸载
timeout=5                         
```

#### 九、测试

重启测试是否完成挂载。