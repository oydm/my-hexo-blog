---
title: Ubuntu 16.04 安装snmp
date: 2018-03-29 16:05:00
categories:
- Linux
tags:
- Linux
---

通过snmp的方式获取linux主机信息进行linux主机的监控

# 安装
需要安装软件有：
- snmpd：snmp服务端软件
- snmp：snmp客户端软件
- snmp-mibs-downloader：用来下载更新本地mib库的软件

ubuntu一键安装命令：
``` bash
sudo apt-get install snmpd snmp snmp-mibs-downloader
```
安装snmp-mibs-downloader的过程中会自动下载mib库，并保存在/usr/share/mibs目录中：
``` bash
root@ubuntu:/home/sky# cd /usr/share/mibs/
root@ubuntu:/usr/share/mibs# ll
total 8
drwxr-xr-x   2 root root 4096 Mar 28 06:13 ./
drwxr-xr-x 142 root root 4096 Mar 28 06:13 ../
lrwxrwxrwx   1 root root   18 Jun  7  2010 iana -> /var/lib/mibs/iana/
lrwxrwxrwx   1 root root   18 Jun  7  2010 ietf -> /var/lib/mibs/ietf/
```
如果没有这写目录文件的话，可以执行命令安装：
``` bash
sudo download-mibs
```
安装完后，系统已经自动开启了这个服务，可以检测下：
``` bash
root@ubuntu:/usr/share/mibs# sudo service snmpd status
● snmpd.service - LSB: SNMP agents
   Loaded: loaded (/etc/init.d/snmpd; bad; vendor preset: enabled)
   Active: active (running) since Wed 2018-03-28 01:24:32 UTC; 5h 1min ago
     Docs: man:systemd-sysv-generator(8)

```
检测服务正常：
``` bash
root@ubuntu:/usr/share/mibs# snmpwalk -v 2c -c public localhost 1.3.6.1.2.1.1.1
iso.3.6.1.2.1.1.1.0 = STRING: "Linux ubuntu 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64"
```
有正常信息返回，安装OK，接下来做一些配置

# 配置
snmp的配置文件位于/etc/snmp/snmpd.conf,在配置之前最好对配置文件先做一个备份：
### 配置节点
修改/etc/snmp/snmpd.conf文件，将下面两行注释掉：
``` bash
#view   systemonly  included   .1.3.6.1.2.1.1
#view   systemonly  included   .1.3.6.1.2.1.25.1
```
并在下面增加一行：
``` bash
view  systemonly  included  .1
```
这样配置是为量获取更多的节点信息。

修改完后，重启snmp服务并测试一下：
``` bash
root@ubuntu:/etc/snmp# sudo service snmpd restart
root@ubuntu:/etc/snmp# snmpwalk -v 2c -c public localhost .1.3.6.1.4.1.2021.4.3.0
iso.3.6.1.4.1.2021.4.3.0 = INTEGER: 2046968
```
OK，没有问题！不过需要注意的是，这里.1.3.6.1.4.1.2021.4.3.0表示的是LInux主机交换空间总量的一个节点，而输出2046968，就说明我们的主机上的交换空间总量大概就是2GB左右。
### 配置MIB库
修改/etc/snmp/snmp.conf配置文件，将mibs : 注释掉：
``` bash
    
# As the snmp packages come without MIB files due to license reasons, loading
# of MIBs is disabled by default. If you added the MIBs you can reenable
# loading them by commenting out the following line.

#mibs :

```
重启snmp服务并测试一下：
``` bash
root@ubuntu:/etc/snmp# sudo service snmpd restart
root@ubuntu:/etc/snmp# snmpwalk -v 2c -c public localhost .1.3.6.1.4.1.2021.4.3.0
UCD-SNMP-MIB::memTotalSwap.0 = INTEGER: 2046968 kB
```

命令中的.1.3.6.1.4.1.2021.4.3.0为OID节点，常用一些节点可以在网上查找:[SNMP监控一些常用OID的总结](https://www.cnblogs.com/aspx-net/p/3554044.html)

### 配置共同体
共同体 类似为密码认证，前面我们在使用snmp的时候命令中有-c public 其中public为默认的共同体，

修改配置文件/etc/snmp/snmpd.conf，将
``` bash
#rocommunity public  localhost
                                                 #  Default access to basic system info
 rocommunity public  default    -V systemonly
                                                 #  rocommunity6 is for IPv6
 rocommunity6 public  default   -V systemonly

```
修改为：
``` bash

#rocommunity public  localhost
                                                 #  Default access to basic system info
 rocommunity sky  default    -V systemonly
                                                 #  rocommunity6 is for IPv6
 rocommunity6 sky  default   -V systemonly

```
可以重启再用命令测试一下，你会发现前面的命令并不能正常的返回信息量，将命令中的共同体修改为我们指定的，比如sky：
``` bash
root@ubuntu:/etc/snmp# snmpwalk -v 2c -c sky localhost  .1.3.6.1.2.1.25.2.2.0
HOST-RESOURCES-MIB::hrMemorySize.0 = INTEGER: 2029876 KBytes
```
### 允许远程访问
默认情况下，snmp服务只是对本地开启，是无法通过远程获取该主机的snmp信息的：
``` bash
root@ubuntu:/etc/snmp# sudo netstat -antup | grep 161 
udp        0      0 127.0.0.1:161           0.0.0.0:*                           23013/snmpd
```
可以看到，161端口只对本机开放（161端口号是snmp服务的端口号），我们需要修改一下，让snmp服务对外开放。

修改/etc/snmp/snmpd.conf配置文件,将下面一行注释掉：
``` bash
agentAddress  udp:127.0.0.1:161
```
同时取消注释：
``` bash
#agentAddress udp:161,udp6:[::1]:161
```
重启服务你会发现短裤已经对外开放了
``` bash
root@ubuntu:/etc/snmp# sudo service snmpd restart
root@ubuntu:/etc/snmp# sudo netstat -antup | grep 161 
udp        0      0 0.0.0.0:161             0.0.0.0:*                           25471/snmpd     
udp6       0      0 ::1:161                 :::*                                25471/snmpd
```

### 远程测试
将命令中的localhost换成你要测试的远程主机ip即可。
