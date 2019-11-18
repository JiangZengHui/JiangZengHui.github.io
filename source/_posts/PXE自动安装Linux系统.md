---
title: PXE自动安装Linux系统
top: false
cover: false
toc: true
mathjax: true
date: 2019-10-19 22:33:26
password:
summary: PXE全称是：Preboot Excution Environment(预启动执行环境)是由Intel公司研发的基于Client/Server的网络模式。
tags: Linux网络运维
categories: Linux
---


#### 一、PXE简介

预启动执行环境（Preboot eXecution Environment，PXE，也被称为预执行环境)提供了一种使用网络接口（Network Interface）启动计算机的机制。

> 这种机制让计算机的启动可以不依赖本地数据存储设备（如硬盘）或本地已安装的操作系统。

PXE当初是作为Intel的有线管理体系的一部分，Intel 和 Systemsoft于1999年9月20日公布其规格(版本2.1)。

> 通过使用像网际协议(IP)、用户数据报协议(UDP)、动态主机设定协定(DHCP)、小型文件传输协议(TFTP)等几种网络协议和全局唯一标识符(GUID)、通用网络驱动接口(UNDI)、通用唯一识别码(UUID)的概念并通过对客户机(通过PXE自检的电脑)固件扩展预设的API来实现目的。

因此它可以用来通过网络从远端服务器下载映像，并由此引导和安装Windows,linux等多种操作系统。

#### 二、环境准备

首先准备至少两台虚拟机，其中一台作为服务器使用。(基本所有配置操作都是在服务器上客户端机器不需要配置。）还需要至少一个安装源（系统安装镜像文件）
1. 将两台虚拟机网卡配置为仅主机模式（主要是和外网隔离，以面影响实体机的DHCP服务）
2. 关闭VMware的DHCP服务
3. 给服务器分配一个IP地址，地址建议为静态地址。配置文件如下：

```
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.11.2
PREFIX=24
GATEWAY=192.168.11.1
NAME=ens33
DEVICE=ens33
ONBOOT=yes
```

4. 编辑Win下的虚拟网卡配置以便和虚拟机可以通信
打开控制面板→网络和共享中心→更改适配器设置
通常仅主机的连接名称为VMnet1
> 地址可以随意分配，但确保此处地址是上一步分配的地址的网关。如果在虚拟机中可以ping通网关则说明配置没有问题。


#### 三、防火墙和SELinux的设置


> 确保SELinux和防火墙处于关闭状态。


将/etc/selinux/config文件中的SELINUX=xxx改为SELINUX=disabled并重启。


> 可以通过命令getenforce查看，如果是disabled或permissive即为关闭状态


执行以下两条命令确保关闭防火墙


```
systemctl stop firewalld
systemctl disable firewalld
```


以上是CentOS 7，如果是CentOS 6的话


```
service iptables stop
chkconfig iptables off
```

#### 四、安装软件包并开启服务

> 出于方便，软件安装全部使用yum进行安装，如果没有yum源的请自行配置。

执行以下命令安装相关软件并启动服务，DHCP由于需要另外修改配置文件先跳过

```
yum install httpd dhcp syslinux tftp-server
systemctl start httpd tftp.socket
systemctl enable dhcpd tftp.socket httpd
```

 -  使用ss -tan 确认TCP80端口已开启
 -  使用ss -unl 确认UDP69端口已开启

####  五、准备安装源

此次实验我们通过HTTP作为安装源，所以，只要将我们准备好的安装镜像挂载到HTTP目录下可以访问即可。具体目录是/var/www/html/ 为了方便扩展还可以在此目录下建立几个文件夹，以存放不同版本的系统文件。

具体实现：
1. 使用mkdir -p /var/www/html/Centos/7创建文件夹
2. 将光盘挂载到/var/www/html/Centos/7目录下
3. 在/etc/fstab文件中添加自动挂载信息，通常为 /etc/sr0 /var/www/html/Centos/7 iso9660 default 0 0
4. 使用df -h确认挂载信息
5. 在主机使用浏览器访问http://192.168.11.2/Centos/7 确认可以看到挂载的安装文件

#### 六、准备自动应答文件

> 自动应答文件是整个环节相当重要的部分（其实每一部分都很很重要）自动应答文件的生成可以通过在图形界面下安装system-config-kickstart包使用这个工具在图形界面下生成，或者系统安装完成后默认在root家目录有一个叫anaconda-ks.cfg的文件，也可以直接修改这个文件。

由于图形界面比较简单，这里直接修改anaconda-ks.cfg文件。

其中以#开头的行表示注释，如果你没有修改过此文件中的内容，那么应该内容如下（中文为后加）：

> 注：此为最小化安装文件内容，如果是图形界面，内容会有些许不同

```
#platform=86, AMD64, or Intel EM64T
#version=DEVEL
# System authorization information
auth --useshadow  --passalgo=sha512
# Install OS instead of upgrade#install是安装，upgrade是升级
install
# Use CDROM installation media#安装介质
cdrom
# Use text mode install#使用图形安装还是文字界面
text
# Firewall configuration#防火墙选项选择关闭
firewall --disabled
firstboot --disable
ignoredisk --only-use=sda
# Keyboard layouts
# old format: keyboard us
# new format:
keyboard --vckeymap=us --xlayouts=''
# System language
lang en_US.UTF-8

# Network information#安装后网络配置信息，可以将onboot改为on
network --bootproto=dhcp --device=ens33 --onboot=off 
        --ipv6=auto --no-activate
network  --hostname=localhost.localdomain
# Reboot after installation#安装之后重启
reboot
# Root password# root账号密码
rootpw --iscrypted $1$HwDDpzbI$JcacPj2.QTbRQgNWUP8hr1
# SELinux configuration#SELinux选项
selinux --disabled
# System services
services --enabled="chronyd"
# Do not configure the X Window System
skipx
# System timezone# 时区
timezone Asia/Shanghai
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
#Clear the Master Boot Record
#zerombr #清除MBR
# Partition clearing information
#clearpart --all --initlabel # 清空磁盘
# Disk partitioning information
#以下三行是分区信息
#part swap --fstype="swap" --size=2048
#part / --fstype="xfs" --size=20480
#part /boot --fstype="xfs" --size=1024

#要安装的包@开头的是包组，没有@的是单独的包
%packages
@core
zsh

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
```

看起来很多内容，也有可能有很多不一样，不过没有关系，许多内容并不需要理解，需要改的地方并不多
大概以下几项修改就可以了：
1. 安装介质即把cdrom改为以下内容

```
url --url="http://192.168.11.2/centos/7/"
```

2. 防火墙和SELinux根据自己的需要选择开启或关闭。
3. 如果没有zerombr和clearpart --all --initlabel，手动添加如果前面有#，则去掉。
4. 去掉分区信息前的#。
5. 根据自己的需要选择需要安装的包，最小化只有一个core即可。
6. 将文件保存之后放到可以被HTTP访问到的地方。

> 如/var/www/html/，确保文件权限为644

7. 在主机访问http://192.168.11.2/anaconda-ks.cfg 如果可以成功访问，说明此步骤OK

手动修改应答文件有时候可能并不是一个明智的选择，如果没有把握，使用图形工具是更好的选择

> 需要注意的是，注意应答文件和要安装系统的对应，比如安装6的系统请使用6生成应答文件


#### 七、配置DHCP服务并启动

默认情况下DHCP服务在安装完成之后配置文件是空的，也因此DHCP服务必须进行一定的配置才可以启动。

> DHCP的服务配置文件路径是:/etc/dhcp/dhcpd.conf。

- 网段：服务器可以分配的地址的网段，可以指定多个
- 掩码：网段对应的掩码
- 默认租期：以秒为单位默认的IP地址的租期
- 最长租期：客户端可以请求一个租期，此项设定用于对此进行限制以秒为单位
- 地址池：服务器在可分配网段中可以分配的IP地址的范围
- DNS：用于客户机从DHCP服务器获取的DNS地址
如下一个示例：

```
# 指定从DHCP服务器获取的DNS地址
option domain-name-servers 114.114.114.114
# 默认IP地址的租期
default-lease-time 600；
# 最长租期
max-lease-time 7200；
# subnet用来指定网段，netmask是掩码
subnet 192.168.11.0 netmask 255.255.255.0
{
    # 使用range指定IP地址池
    range 192.168.11.3 192.168.11.124;
}
对于一个DHCP服务器以上内容足够了
next-server 192.168.11.2
filename "pxelinux.0"
next-server用来指定TFTP服务器的位置
filename用来指定要访问TFTP服务器上的哪个文件
也就是说在这里我们的DHCP服务器的配置文件是这样的

option domain-name-servers 114.114.114.114
default-lease-time 600；
max-lease-time 7200；
subnet 192.168.11.0 netmask 255.255.255.0
{
    next-server 192.168.11.2;
    filename "pxelinux.0";
    range 192.168.11.3 192.168.11.124;
}
```

接下来使用systemctl start dhcpd启动dhcp服务，使用ss -unl查看67端口是会否已经开启。

#### 八、必要文件复制到相关目录

将相关文件复制到TFTP共享目录。

> TFTP的共享目录在/var/lib/tftpboot/下，考虑到灵活性依旧可以给不同版本的系统单独的目录

- 这里创建一个7的目录
复制光盘目录中isolinux下的initrd.img和vmlinuz到/var/lib/tftpboot/7目录下
- 在共享目录下创建一个名为pxelinux.cfg的文件夹并将光盘目录中isolinux下的isolinux.cfg复制到pxelinux.cfg目录下并且命名为default
将/usr/share/syslinux/menu.c32复制到/var/lib/tftpboot目录下，此文件是菜单背景文件
- 将/usr/share/syslinux/pxelinux.0复制到/var/lib/tftpboot目录下

如果只有一个系统，应该有5个文件两个目录，结构如下

```
[root@DHCP_Svr]/var/lib/tftpboot# tree
.
├── 7
│   ├── initrd.img
│   └── vmlinuz
├── menu.c32
├── pxelinux.0
└── pxelinux.cfg
    └── default
```

#### 九、修改启动菜单以及配置文件

修改default文件,从光盘复制过来的文件有很多内容，不够大部分我们并不需要，参考下面的内容就可以了。

```
#就是刚才复制到菜单文件
default menu.c32
#超时时间，就是菜单倒计时
timeout 600
display boot.msg

# 启动菜单的具体配置
# menu label 用来指定菜单名称，可以自定义，^符号用来确定光标位置，
# 同时其后的字母也是调到对应菜单的快捷键
# kernel指定内核文件路径，由于我们放在了文件夹中所以路径是7/vmlinuz
# 指定initrd的路径，以及ks应答文件文件的路径
# 务必确保应答文件可以访问
label linux
  menu label ^Install CentOS 7
  kernel 7/vmlinuz
  append initrd=7/initrd.img ks=http://192.168.11.2/anaconda-ks.cfg

# 本地硬盘启动
# menu default表示此项菜单为默认菜单，建议将本地启动作为默认启动
label local
  menu default
  menu label Boot from ^local drive
  localboot 0xffff

menu end
```

#### 十、测试

- 将另一台虚拟机网卡同样配置为仅主机，网卡使用自动获取IP。
- 确认安装源，以及应答文件可以访问，且default文件中路径配置正确。
- 确认防火墙以及SELinux处于关闭状态。

确认无误的话就可以开机进行测试了。