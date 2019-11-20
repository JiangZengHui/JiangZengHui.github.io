---
title: 网络存储服务器IP-SAN的搭建
top: false
cover: false
toc: true
mathjax: true
date: 2019-11-2 21:33:36
password:
summary: IP SAN简称SAN（Storage Area Network），中文意思存储局域网络
tags: Linux网络运维
categories: Linux
---

#### 一、IP SAN简介
IP SAN简称SAN（Storage Area Network），中文意思存储局域网络，IP SAN使存储空间得到更加充分的利用，并使得安装和管理更加有效。

SAN是一种将存储设备、连接设备和接口集成在一个高速网络中的技术。SAN本身就是一个存储网络，承担了数据存储任务，SAN网络与LAN业务网络相隔离，存储数据流不会占用业务网络带宽。

#### 二、服务端配置

##### 1. 安装ISCSI服务器端
 

```cpp
# yum -y install scsi-target-utils
```

##### 2.  启动服务器端
 

```cpp
# service tgtd start
# chkconfig tgtd on
# netstat -tnlp | grep 3260
```

##### 3. 建分区

```cpp
# fdisk -l |grep "Disk"
# fdisk  /dev/sdb
```

##### 4. 共享分区

```cpp
# vi /etc/tgt/targets.conf   
<target iqn.2016-08.cn.node01.www:target4_scan> 
        backing-store /dev/sdb1    
        initiator-address 192.168.25.12      
        initiator-address 192.168.25.13
        vendor_id node        
        product_id target4      
</target>
```

#### 二、客户端配置

##### 1. 安装ISCSI客户端

```cpp
# yum -y install iscsi-initiator-utils
# iscsiadm -m discovery -t sendtargets -p 192.168.25.11   
# /etc/init.d/iscsid status  
# tree /var/lib/iscsi/        
# /etc/init.d/iscsid  start        
# /etc/init.d/iscsi  start 
```

##### 2. 寻找ISCS设备

```cpp
# iscsiadm -m discovery -t st -p 192.168.25.15:3260
```

##### 3. 登录到iSCSI设备

```cpp
# iscsiadm -m node -T iqn.2015-03.com.xsl.www:
  xsl.disk1 -p 192.168.25.15:3260 -l
```

##### 4.验证iscsi设备

```cpp
#fdisk -l
```

##### 5.测试

```cpp
# iscsiadm -m discovery -t st -p 192.168.25.15:3260
# iscsiadm -m node -T iqn.2015-03.com.xsl.www:
  xsl.disk1 -p 192.168.25.15:3260 -l
```
