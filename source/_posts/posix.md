---
title: POSIX函数简介
date: 2017-11-02 11:27:09
categories:
- PHP
---

# POSIX:
## 简介：
该模块包含了定义在 IEEE 1003.1(POSIX.1)标准文档里的函数的接口，通过其它手段无法访问。  
警告：
通过POSIX函数，可以检索很多敏感数据，例如：posix_getpwnam()以及其它函数。当开启了安全模式，POSIX函数并不会执行任何的权限检测。因此，当正在上述的环境下操作，强烈建议关闭POSIX扩展(在配置行中使用 '--disable-posix')。  
## 安装：  
POSIX函数默认是启用的，可通过 '--disable-posix' 来禁用POSIX函数
预定义常量，分3大类：  

 1.posix_access_constants - 权限相关，php5.1.0开始支持:  
>POSIX_F_OK - 检查文件是否存在  
>POSIX_R_OK - 检查文件是否存在，且具有 '读' 权限  
>POSIX_W_OK - 检查文件是否存在，且具有 '写' 权限  
>POSIX_X_OK - 检查文件是否存在，且具有 '执行' 权限

2.posix_mknod_constants - 文件类型，php5.1.0开始支持:  
    

>POSIX_S_IFBLK - 块特殊文件  
>POSIX_S_IFCHR - 字符特殊文件  
>POSIX_S_IFIFO - FIFO(pipe-管道)特殊文件  
>POSIX_S_IFREG - 普通文件  
>POSIX_S_IFSOCK - socket

3.posix_setrlimit_constants - php7.0.0开始支持:  
    

> 你不妨看一下下面参考页，关于你的操作系统的setrlimit()的注意点，有关于实现POSIX的limits的差异的解释，甚至是跨操作系统的声明。
> 
>POSIX_RLIMIT_AS - 进程地址空间的最大尺寸，单位是bytes。也可查看PHP的 'memory_limit' 配置指令  
>POSIX_RLIMIT_CORE - 核心文件的最大尺寸。如果设置为0，将不会生成核心文件  
>POSIX_RLIMIT_CPU - 进程可使用的CPU最大时间，单位是秒。当达到软限制(soft limit)，将发送一个 'SIGXCPU' 信号，这个信号可以被 'pcntl_signal()' 捕获。依赖于操作系统，每秒都会发送额外的 'SIGXCPU'
> 信号，直到达到硬限制(hard limit)，基于这点，会发送一个无法捕获的 'SIGKILL' 信号。也可查看
> 'set_time_limit()'  
>POSIX_RLIMIT_DATA - 进程数据段的最大尺寸，单位是bytes。这基本不会对PHP的执行造成任何影响，除非使用了一个叫做 'brk()' 或 'sbrk()'
> 的扩展  
>POSIX_RLIMIT_FSIZE - 进程可以创建的文件的最大尺寸，单位是bytes  
>POSIX_RLIMIT_LOCKS - 进程可以创建的最大的锁定数量。仅支持非常老版的linux内核  
>POSIX_RLIMIT_MEMLOCK - 内存中，可以锁定的最大字节数  
>POSIX_RLIMIT_MSGQUEUE - 可以分配给 POSIX 消息队列的最大字节数。PHP不支持POSIX 消息队列，因此，这个限制没有任何影响，除非，你使用了一个实现了支持 'POSIX_RLIMIT_MSGQUEUE' 的扩展  
>POSIX_RLIMIT_NICE - 进程可以设置 'renice'(linux进程的优先级之类的) 的最大值。值可以被设置为：20-我们设置的值，作为资源限制，不能设置为负  
>POSIX_RLIMIT_NOFILE - 进程可以打开的 >(大于)最大文件描述符数字的值。  
>POSIX_RLIMIT_NPROC - 进程的真实用户ID可以创建的进程(和线程、或者线程，在一些操作系统上)的最大个数。  
>POSIX_RLIMIT_RSS - 进程的常驻集合的最大尺寸，单位是 pages  
>POSIX_RLIMIT_RTPRIO - 通过 'sched_setscheduler()' 和 'sched_setparam()' 系统调用，可以设置的最大真实时间优先。  
>POSIX_RLIMIT_RTTIME - 如果使用真实的时间调度，在不进行阻塞的系统调用下，进程可以消耗掉最大CPU时间，单位是微秒  
>POSIX_RLIMIT_SIGPENDING - 进程的真实用户ID，可以设置的信号队列的最大个数  
>POSIX_RLIMIT_STACK - 进程栈的最大尺寸，单位是bytes  
>POSIX_RLIMIT_INFINITY - 用于指明资源大小不受限制(给资源限制设置了一个无限大值)。

# 函数
## posix_access(string $file[, int $mode = POSIX_F_OK])  
查看用户对文件是否具有指定的权限  

> 参数：  
>$file - 测试的文件名  
>$mode - 权限，包含：POSIX_F_OK, POSIX_R_OK, POSIX_W_OK, POSIX_X_OK的一个或多个。

  
## posix_ctermid()  
返回当前进程所在的当前控制终端的路径名  

> 返回值：   成功时，返回路径名。否则返回false，并设置错误号。可通过 'posix_get_last_error()' 来获取

## posix_errno() - posix_get_last_error()别名  
## posix_get_last_error()  
检索最后的posix函数调用失败，返回的错误号。错误号关联的错误消息，可通过 'posix_strerror()' 来获取  
## posix_strerror(int $errno)  
通过给定的错误号，返回关联的POSIX系统错误消息  
## posix_getcwd()  
获取当前脚本的工作目录的绝对路径  
## posix_getegid()  
返回当前进程的有效用户组ID  
## posix_geteuid()  
返回当前进程的有效用户ID  
## posix_getgid()  
返回当前进程的真实用户组ID  
## posix_getuid()  
返回当前进程的真实用户ID  
## posix_getgrgid(int $gid)  
通过传入组ID，获取给定的用户组的相关信息  
## posix_getgrnam(string $name)  
通过传入组名称，获取给定的用户组的相关信息  
## posix_getgroups()  
获取当前进程的用户组集合  
返回值：  
返回一个索引数组，包含组id的集合  
## posix_getlogin()  
返回拥有当前进程的用户的登陆名   
> 示例：  
>    <?php  
>   echo posix_getlogin(); // apache  
>    ?>

  
## posix_getpgid(int $pid)  
获取指定进程的进程组标识符(进程组id)，返回整数  

> 注意：  
>该函数不是POSIX函数，但是常见于BSD和System V的系统上。如果系统不支持该函数，在编译时就不会被包含进来。应该提前使用 'function_exists()' 检查，存在再使用

 
## posix_getpgrp()  
获取当前进程的进程组标识符(进程组id)，返回整数  
可查看：POSIX.1 和 POSIX系统上的getpgrp(2) 帮助手册，获取关于进程组的更多信息  
## posix_getpid()  
获取当前进程的进程标识符(进程id)  
## posix_getppid()  
获取当前进程的父进程标识符(父进程id)  
## posix_getpwnam(string $username)  
通过用户名，获取给定用户的信息  
  

> 返回值：  
>    成功，返回一个关联数组，下标如下，失败返回false：  
>    name - 是一个短的、通常少于16个字符，非真实的、全名。应该同调用函数时，传递的$username参数一致，截断多余的字符  
>    passwd - 返回加密后的用户密码的字符串。通常，例如系统的shadow密码，使用 '*' 代替  
>    uid - 用户ID  
>    gid - 用户组ID。使用 posix_getgrgid() 获取用户组名和它的成员列表  
>    gecos - 一个过时的元素，包含了 ',' 分隔的用户的全名、办公室电话、办公室号码以及家庭电话号码。大多数的系统上，只有用户的全名有效。  
>    dir - 用户家目录的绝对路径  
>    shell - 可执行的用户的默认shell的绝对路径  
>示例：  
>    <?php  
>   $userinfo = posix_getpwnam('tom');  
>   print_r($userinfo);  
>    ?>  
>输出：  
>    Array(  
>   [name]    => tom  
>   [passwd]  => x  
>   [uid]=> 10000  
>   [gid]=> 42  
>   [gecos]   => "tom,,,"   // ',' 分隔  
>   [dir]=> "/home/tom"  
>   [shell]   => "/bin/bash"  
>    )

## posix_getpwuid(int $uid)  
通过用户id，获取给定用户的信息  
## posix_getrlimit()  
返回一个关于当前资源的软限制和硬限制的信息数组  
>每个资源有一个关联的软限制和硬限制。  
>软限制－查看linux系统  
>硬限制－查看linux系统
>一个无特权的进程，可能只能设置它的软限制为：0-硬限制大小，并且必须低于硬限制的值。  
>返回值：  
>    返回一个关联数组，下标为定义的各种限制。每个限制都有一个软限制和硬限制。  
>    core - 核心文件的最大尺寸。当为0，不会创建核心文件。核心文件大于该设定值，将会被截断  
>    totalmem - 进程的内存最大值，单位为bytes  
>    virtualmem - 进程的虚拟内容的最大值，单位为bytes  
>    data - 进程的数据段的最大值，单位为bytes  
>    stack - 进程栈的最大值，单位为bytes  
>    rss - RAM中常驻的虚拟页面的最大个数  
>    maxproc - 可被调用进程的真实用户ID创建的最大进程数量  
>    memlock - 在RAM中，可被锁定的内存的最大字节数  
>    cpu - 进程允许使用的最大CPU时间  
>    filesize - 进程可以创建的文件的最大尺寸，单位是bytes  
>    openfiles - 进程可以打开的最大文件数量  
>示例：  
>    <?php  
>   $limits = posix_getrlimit();  
>   print_r($limits);  
>    ?>  
>输出：  
>    Array(  
>   [soft core] => 0  
>   [hard core] => unlimited  
>   [soft data] => unlimited  
>   [hard data] => unlimited  
>   [soft stack] => 8388608  
>   [hard stack] => unlimited  
>    )

## posix_getsid(int $pid)  
返回指定进程的session ID。进程的session ID是会话领导者(session leader)的进程组id  
## posix_initgroups(string $name, int $base_group_id)  
对指定的用户，计算其组访问列表  
 

>  参数：  
>  $name - 指定的用户名  
>  $base_group_id - 密码文件里的组ID  
## posix_isatty(mixed $fd)  
检查文件描述符是否是一个有效的终端类型的设置(是否是tty)  
>参数：  
> $fd - 文件描述符，期望是一个文件资源或一个整型。整型将被假定为可以直接传递到基础系统调用的文件描述符。几乎在所有情况下，提供的是一个文件资源。

 
## posix_kill(int $pid, int $sig)  
给指定的进程发送一个$sig指定的信号！  
参数：  

>$pid - 进程id  
> $sig - PCNTL信号预定义常量  
## posix_mkfifo(string $pathname, int $mode)  
创建一个特殊的FIFO文件，存在于文件系统，并且作为进程的双向通信桥梁
> 参数：  
> $pathname - FIFO文件(管道)  
> $mode - 必须是8进制格式。新创建的FIFO的权限，也依赖于当前的umask()设置。新创建的文件权限是(mode & ~umask)

## posix_mknod(string $pathname, int $mode[, int $major = 0[, int $minor = 0]])  
创建一个特殊的或者一般的文件  

> 参数：
$pathname - 创建的文件   
$mode - 这个参数由文件类型(POSIX_S_IFREG,POSIX_S_IFCHR, POSIX_S_IFBLK,POSIX_S_IFIFO,POSIX_S_IFSOCK其中一个)和访问权限(0664等)，按位或组成。
>$major -主设备内核标识符(当使用S_IFCHR或S_IFBLK时，需要传递该参数)   $minor - 监控设备内核标识符

## posix_setegid(int $gid)  
设置当前进程的有效组ID。这是个特权函数，需要操作系统上具有特殊权限(通常是root权限)，才能执行该函数。  
## posix_seteuid(int $uid)  
设置当前进程的有效用户ID。这是个特权函数，需要操作系统上具有特殊权限(通常是root权限)，才能执行该函数。  
## posix_setgid(int $gid)  
设置当前进程的真实用户组ID。这是个特权函数，需要操作系统上具有特殊权限(通常是root权限)，才能执行该函数。函数调用的适当的顺序是：首先调用 posix_setgid()，最后调用 posix_setuid()。  
注意：如果是超级用户调用，也会设置有效用户组ID  
## posix_setpgid(int $pid, int $pgid)  
设置指定进程的进程组ID  
##posix_setrlimit(int $resource, int $softlimit, int $hardlimit)  
设置给定系统资源的软限制和硬限制。  

> 参数：   $resource - 是posix_setrlimit_constants预定义常量   $softlimit -
> 软限制，任意设置或者 POSIX_RLIMIT_INFINITY - 无限大   $hardlimit - 硬限制，任意设置或者
> POSIX_RLIMIT_INFINITY - 无限大

## posix_setsid()  
设置当前进程为session leader(会话领导者)  
## posix_setuid(int $uid)  
设置当前进程的真实用户ID。这是个特权函数，需要操作系统上具有特殊权限(通常是root权限)，才能执行该函数。  
## posix_times()  
获取当前CUP使用信息  
 

> 返回值：  
>  返回一个关联数组  
> ticks - 重启到现在，已经过去的 clock ticks 个数  
> utime - 当前进程使用的用户时间  
> stime - 当前进程使用的系统时间  
> cutime - 当前进程和子进程使用的用户时间  
> cstime - 当前进程和子进程使用的系统时间  
> 警告：

## posix_ttyname($mixed $fd)  
返回当前打开的文件描述符所在的终端设备的绝对路径  

> 参数：  
>$fd - 文件描述符，期望是一个文件资源或一个整型。整型将被假定为可以直接传递到基础系统调用的文件描述符。几乎在所有情况下，提供的是一个文件资源。

## posix_uname()  
获取系统相关信息。  

> 返回值：返回一个关于系统信息的关联数组  
>sysname - 操作系统名称(例如：Linux)  
>nodename - 系统名称(例如：valiant)  
>release - 操作系统的发布版(例如：2.6.15-1-686)  
>version - 操作系统版本(例如：#4 Tue Jul 20 17:01:36 MEST 1999)  
>machine - 系统平台(例如：i586)  
>domainname - DNS域名(例如：baidu.com)