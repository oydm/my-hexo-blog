---
title: linux的监控管理服务器命令
date: 2018-03-27 16:03:45
categories:
- Linux
tags:
- Linux
---

环境：VMware虚拟机

系统：Ubuntu 16.04.4 LTS

## 查看CPU相关数据

### lscpu
查看CPU信息
``` bash
root@ubuntu:~# lscpu
Architecture:          x86_64           #架构
CPU op-mode(s):        32-bit, 64-bit   
Byte Order:            Little Endian
CPU(s):                4                #逻辑CPU颗数
On-line CPU(s) list:   0-3
Thread(s) per core:    1        #每个cpu核只能支持一个线程，即不支持超线程
Core(s) per socket:    2        #每个cpu核数
Socket(s):             2        #总共有2一个cpu
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 42
Model name:            Intel(R) Core(TM) i3-2130 CPU @ 3.40GHz  
Stepping:              7
CPU MHz:               3400.000
BogoMIPS:              6800.00
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              3072K
NUMA node0 CPU(s):     0-3

```
### uptime
显示系统已经运行了多长时间,它依次显示下列信息：当前时间、系统已经运行了多长时间、目前有多少登陆用户、系统在过去的1分钟、5分钟和15分钟内的平均负载。
``` bash
root@ubuntu:~# uptime
 03:16:37 up  1:53,  2 users,  load average: 1.04, 1.08, 1.02

```
### top
实时动态地查看系统的整体运行情况
``` bash
top - 03:20:13 up  1:56,  2 users,  load average: 1.16, 1.13, 1.04
Tasks: 205 total,   2 running, 203 sleeping,   0 stopped,   0 zombie
%Cpu(s): 24.0 us,  1.7 sy,  0.0 ni, 74.2 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem :  2029876 total,   206168 free,   465472 used,  1358236 buff/cache
KiB Swap:  2046968 total,  2036916 free,    10052 used.  1340344 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                       
 40224 root      20   0  207100 172196  20516 R  87.4  8.5   0:02.64 cc1plus                                                                       
   924 root      20   0  850076  42460  10336 S   1.7  2.1   3:58.94 python                                                                        
 13824 root      20   0       0      0      0 S   0.7  0.0   0:06.23 kworker/u256:0                                                                
     7 root      20   0       0      0      0 S   0.3  0.0   0:38.89 rcu_sched                                                                     
    32 root      39  19       0      0      0 S   0.3  0.0   0:00.84 khugepaged                                                                    
  1468 snmp      20   0   59652   6588   2288 S   0.3  0.3   0:05.79 snmpd                                                                         
 39936 root      20   0   41808   3800   3104 R   0.3  0.2   0:00.08 top                                                                           
     1 root      20   0   37884   5136   3600 S   0.0  0.3   0:04.00 systemd
     ......
```
- 其中第一行数据和uptime 命令一致
- Tasks 进程信息分别为总进程数、正在运行进程数、睡眠的进程数、停止的进程数、冻结进程数
- CPU信息百分比 分别为us用户空间占用CPU、sy内核空间占用CPU、ni用户进程空间内改变过优先级的进程占用CPU、id空闲CPU、wa等待输入输出的CPU时间、
- 物理内存信息 总内存量、空闲内存量、已使用内存量、缓存内存量
- 虚拟交换内存信息
 
### htop
为top的进阶版，htop更加人性化。它可让用户交互式操作，支持颜色主题，可横向或纵向滚动浏览进程列表，并支持鼠标操作

![htop](http://i1.bvimg.com/638560/d4db56f74f184cf2.png)

上面左上角显示CPU、内存、交换区的使用情况，右边显示任务、负载、开机时间，下面就是进程实时状况。

## 内存
### free
``` bash
root@ubuntu:~# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.9G        409M        1.0G        3.4M        527M        1.3G
Swap:          2.0G         19M        1.9G

```

## 磁盘

### df
磁盘使用情况
``` bash
root@ubuntu:~# df
Filesystem     1K-blocks    Used Available Use% Mounted on
udev              996488       0    996488   0% /dev
tmpfs             202988    5960    197028   3% /run
/dev/sda1       50487988 6914880  40985360  15% /
tmpfs            1014936      16   1014920   1% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
tmpfs            1014936       0   1014936   0% /sys/fs/cgroup
tmpfs             202988       0    202988   0% /run/user/1000
```
### iotop
实时动态地查看系统的读写情况
``` bash
Total DISK READ :       3.67 K/s | Total DISK WRITE :       3.67 K/s
Actual DISK READ:       3.67 K/s | Actual DISK WRITE:      29.36 K/s
   TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                             
 50899 be/4 root        3.67 K/s    0.00 B/s  0.00 %  1.14 % [kworker/u256:1]
   965 be/4 root        0.00 B/s    3.67 K/s  0.00 %  0.00 % python main.pyc 8888
     1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % init noprompt
     2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]
     3 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/0]
     5 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/0:0H]
     7 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_sched]

```
## 网络
### iftop
类似于top的实时流量监控工具。可以用来监控网卡的实时流量（可以指定网段）、反向解析IP、显示端口信息等
![iftop](http://i1.bvimg.com/638560/68f62f83340b2740.png)

界面上面显示的是类似刻度尺的刻度范围，为显示流量图形的长条作标尺用的。 
中间的<= =>这两个左右箭头，表示的是流量的方向。 
TX：发送流量 
RX：接收流量 
TOTAL：总流量 
Cumm：运行iftop到目前时间的总流量  
peak：流量峰值 
rates：分别表示过去 2s 10s 40s 的平均流量
### nethogs 
监控进程流量
![nethogs](http://i4.bvimg.com/638560/3b08bc86a5190dbd.png)

