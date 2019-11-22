---
title: DNS服务器搭建
top: false
cover: false
toc: true
mathjax: true
date: 2019-11-9 11:46:24
password:
summary: DNS是进行域名和与之相对应的IP地址转换的服务器。
tags: Linux网络运维
categories: Linux
---

#### 一、DNS简介
DNS（Domain Name Server，域名服务器）是进行域名(domain name)和与之相对应的IP地址 (IP address)转换的服务器。

DNS中保存了一张域名(domain name)和与之相对应的IP地址 (IP address)的表，以解析消息的域名。 域名是Internet上某一台计算机或计算机组的名称，用于在数据传输时标识计算机的电子方位（有时也指地理位置）。

域名是由一串用点分隔的名字组成的，通常包含组织名，而且始终包括两到三个字母的后缀，以指明组织的类型或该域所在的国家或地区。

####  二、DNS原理
域名解析主要有三种方法:

1. host 表用在本机上面 hosts
2. NIS（网络信息服务）主要用在小型网络，并且已大部分不在使用
3. DNS域名服务，分层分布式数据库来处理IP地址和域名的转换。

> DNS组成：DNS域名+DNS服务器+解析器

解析过程：

1. 本地解析（使用之前的缓存信息，或者本地HOSTS文件）
2. 直接解析（直接有所设定的DNS服务器解析）
3. 递归查询（由DNS服务器代表客户机向其他DNS 服务器查询）
4. 迭代查询（DNS向客户机返回可以解析的其他DNS服务器）

#### 三、DNS软件的安装

```cpp
yum install -y bind
rpm -ql bind //查看没有安装
```

> DNS相关配置文件:
> 1. /etc/named.conf 是BIND的核心配置文件，它包含了BIND 的基本配置，但其并不包括区域数据。
> 2. /var/named/目录是DNS数据库文件存放目录，每一个域文件都放在这里。

#### 四、配置文件参数
在/etc/named.conf中大致分为针对全局生效，存放常规设置的options部分；以及针对某个区域，由各个区域设置的zone部分。另外还有logging（日志选项）和acl（访问控制列表）可选。

去除注释后的/etc/named.conf配置文件如下：


```cpp
[root@jzh ~]# cat /etc/named.conf
options {
//指定BIND通过哪些网络接口和哪个端口来接收客户端请求。
//如果省略端口号，则使用默认端口53。127.0.0.1允许接收来自本地的请求。
//如果完全省略此项或网络接口是any，则默认使用所有端口。
        listen-on port 53 { 127.0.0.1; };
//指定BIND通过哪个端口监听IPv6客户端请求。IPv6，
//服务器只接受通配符地址，any和none。
        listen-on-v6 port 53 { ::1; };
//BIND可以在该目录中找到包含区域数据的文件
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
//定义客户端可以自此发送DNS请求的网络。
//可以用地址信息（192.168.5.101/24）替换或使用any替换
        allow-query     { localhost; };
        recursion yes;
        dnssec-enable yes;
        dnssec-validation yes;
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};
 
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
 
zone "." IN {
        type hint;　　//指定区域类型
        file "named.ca";//区域文件，需要手动创建。
        				//范例文件为named.localhost
};
 
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

type字段指定区域的类型，对于区域的管理至关重要，一共分为5种：

- master：主DNS服务器，拥有区域数据文件，并对此区域提供管理数据。

- slave：辅助DNS服务器，拥有主DNS服务器的区域数据文件的副本，服务器DNS服务器会从主DNS服务器同步所有区域数据。

- stub：stub区域与slave区域类似，但只复制主DNS服务器上的NS记录，而不像slave会复制所有区域数据

- forward：转发配置域

- hint：根域服务器的初始化使用的参数

#### 五、搭建DNS服务器

##### 1、备份DNS服务器的配置文件/etc/named.conf

```cpp
[root@jzh ~]# cp /etc/named.conf /etc/named.conf.bak
```

##### 2、修改DNS服务器的配置文件/etc/named.conf

```cpp
[root@jzh ~]# cat /etc/named.conf
options {
        listen-on port 53 { any; };　　//修改为
        listen-on-v6 port 53 { any; };　　//修改为any
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { any; };
        recursion yes;
        dnssec-enable yes;
        dnssec-validation yes;
        dnssec-lookaside auto;　　//添加此行
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};
 
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
 
zone "test.cn" IN {　　//区域名称改为test.cn
        type master;　　//区域类型为主DNS服务器
        file "test.cn.zone";　　//区域文件，需要手动创建。范例文件为named.localhost
};
 
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

#####  3、创建区域文件，修改权限，并进行修改


```cpp
//-a选项保留源文件的权限
[root@jzh ~]# cp -a 
/var/named/named.localhost /var/named/test.cn.zone
-rw-r----- 1 root named 257 5月 20 17:13 /var/named/test.cn.zone
[root@jzh ~]# vim /var/named/test.cn.zone
$TTL 1D　　//设置有效地址解析记录的默认缓存时间，默认为1天
//test.cn.也可以使用@，表示当前的域
//dns.test.cn.该域的主域名
//root.test.cn.管理员邮件地址
test.cn.        IN SOA  dns.test.cn. root.test.cn. (
//更新序列号，用于标示数据库的变换，可以在10位以内。如果存在辅助DNS区域，建议每次更新完数据库，手动加1
                                        0       ; serial
//刷新时间，从域名服务器更新该地址数据库文件的间隔时间，默认为1天
                                        1D      ; refresh
//重试延时，从域名服务器更新地址数据库失败后，等待多长时间，默认为1小时
                                        1H      ; retry
//到期失效时间，超过改时间人无法更新地址数据库，那么不再尝试，默认为1周
                                        1W      ; expire
//无效地址解析记录默认缓存时间（该数据库中不存在的地址），默认为3小时。
                                        3H )    ; minimum
test.cn.        NS      dns.test.cn.
dns.test.cn.    A       192.168.5.101
www.test.cn.    A       192.168.5.101
cname.test.cn.  CNAME   www.test.cn.
```

#####  4、重启DNS服务器

```cpp
[root@jzh ~]# systemctl restart named
```


##### 5、测试主机上修改DNS地址和网卡信息
 	

```cpp
[root@jzh ~]# vim /etc/resolv.conf
# Generated by NetworkManager
nameserver 192.168.5.101
[root@jzh ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="yes"
NAME="ens33"
UUID="a12d9bb2-85cb-4db0-9b94-e4ed3baa6470"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.5.102"
PREFIX="24"
GATEWAY="192.168.5.2"
DNS1="192.168.5.101"<br>[root@jzh ~]# systemctl restart network
```

##### 6、测试

```cpp
[root@jzh ~]# ping www.test.cn
PING www.test.cn (192.168.5.101) 56(84) bytes of data.
64 bytes from jzh.cn (192.168.5.101): icmp_seq=1 ttl=64 time=2.73 ms
64 bytes from jzh.cn (192.168.5.101): icmp_seq=2 ttl=64 time=0.423 ms
^C

--- www.test.cn ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.423/1.581/2.739/1.158 ms
[root@jzh ~]# ping cname.test.cn
PING www.test.cn (192.168.5.101) 56(84) bytes of data.
64 bytes from jzh.cn (192.168.5.101): icmp_seq=1 ttl=64 time=4.39 ms
64 bytes from jzh.cn (192.168.5.101): icmp_seq=2 ttl=64 time=0.414 ms
^C

--- www.test.cn ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.414/2.405/4.396/1.991 ms
[root@jzh ~]# ping dns.test.cn 
PING dns.test.cn (192.168.5.101) 56(84) bytes of data.
64 bytes from jzh.cn (192.168.5.101): icmp_seq=1 ttl=64 time=2.81 ms
64 bytes from jzh.cn (192.168.5.101): icmp_seq=2 ttl=64 time=0.388 ms
^C

--- dns.test.cn ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.388/1.601/2.814/1.213 ms
```
