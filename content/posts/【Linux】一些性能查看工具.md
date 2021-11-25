---
title: "【Linux】一些性能查看工具"
date: 2021-11-11T16:43:45+08:00
draft: true
tags: [性能, 工具]
catagories: [Linux]
---

本文介绍一些Linux的查看CPU、IO、网络等负载的基础命令。查看性能负载的好工具很多，但是工作中难免会遇到一些裸机，这时各版服务器系统，都默认安装的基础软件就可以“救命”。

### procps-ng包

procps包提供了大家都熟悉的free、ps、top、uptime，

还有一些可能不常用的vmstat、watch、w、pmap、sysctl。还有一些实在不常用，不详细介绍了。



### coreutils包

GNU提供的基本工具，很多基本命令都来自这个包：cat、chmod、chown、cp、rm、mv、ls、ln、link、mkdir、tail、uname、hostname、tty、touch、test、seq、tee、wc、du、df、pwd等等。



### iproute包

一个网络管理工具包合集，取代先前的net-tools(ifconfig，route，ifup，ifdown，netstat)。

可以在系统中查看有哪些命令rpm -ql iproute|grep bin





### 具体介绍

#### 辅助

##### ps

##### https://blog.csdn.net/u011641865/article/details/71435510

常用的组合

```
# 寻找某个命令的进程
ps -ef|grep xxx
# 显示一些用户、cpu、内存、进程状态信息
ps aux

```



#### 综合工具

##### top

```
top - 20:50:07 up  6:38,  2 users,  load average: 0.29, 0.14, 0.11
Tasks: 125 total,   1 running, 124 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 97.0 id,  0.0 wa,  0.0 hi,  3.0 si,  0.0 st
KiB Mem :  3863568 total,  2506548 free,   752544 used,   604476 buff/cache
KiB Swap:  2098172 total,  2098172 free,        0 used.  2802372 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                    
     1 root      20   0  128392   6980   4168 S   0.0  0.2   0:02.31 systemd                                    
     2 root      20   0       0      0      0 S   0.0  0.0   0:00.01 kthreadd                                   
     3 root      20   0       0      0      0 S   0.0  0.0   0:01.47 ksoftirqd/0
```

第一行的load average，三个数字依次为1 分钟、5 分钟、15 分钟的平均负载

Tasks中有4中类型进程 running表示正在使用cpu的进程，受内核数量限制，重点注意zombie表示停止了，但还没有被父进程回收资源。

%Cpu指标为百分比：us  sy  ni  id  wa  hi  si  st 依次为

user  system  nice  idle  io-wait   hardware interrupts   software interrupts   stolen

用户态 核态 低优先级用户态 空闲 等待IO 硬中断 软中断  其他虚拟机



##### vmstat

查看内存

```shell
# 2秒间隔 采集3次
vmstat 2 3
# -a显示active and inactive memory  -n只显示一次表头
vmstat -a -n 2 3
# -m slab信息
vmstat -mn
# -s打印当前系统时间和内存统计  -S可控制显示单位 K或M
vmstat -s -S M
```

解释一下表头

procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st

r 等待任务数

b 等待IO的进程数量

swpd 在用的虚拟内存

free 空闲内存

buff 已用的设备读写缓冲

cache已用的文件系统cache

inact 非活跃表明可回收

active 活跃内存

si 从交换区写入内存的速度kb/s

so 从内存写入交换区的速度kb/s

bi 磁盘读取块数的速度block/s, block一般为1024bytes

bo 磁盘写入速度

in 每秒中断数

cs 每秒上下文切换数（越小越好）

us、sy、id、wa在top的cpu指标里已介绍



vmstat除了看内存还可以看硬盘IO

```shell
# 3秒 2次
vmstat -d 3 2
# 磁盘汇总
vmstat -D
```



#### CPU相关

##### uptime

```shell
20:50:41 up  6:38,  2 users,  load average: 0.16, 0.12, 0.11
```

update显示的平均负载和top中的load average相同



#### 内存相关

##### free

```shell
# 一些标识：-h 人类可读  -w 宽打印（buff和cache分开）
# -s 3 3秒打印一次
# -c 3 打印3次
$ free -h -s 3
```

**Mem** 行(第二行)是内存的使用情况。
**Swap** 行(第三行)是交换空间的使用情况。
**total** 列显示系统总的可用物理内存和交换空间大小。
**used** 列显示已经被使用的物理内存和交换空间。
**free** 列显示还有多少物理内存和交换空间可用使用。
**shared** 列显示被共享使用的物理内存大小。
**buff/cache** 列显示被 buffer 和 cache 使用的物理内存大小，是已被使用的内存。
**available** 列显示还可以被应用程序使用的物理内存大小。



#### 网络相关

##### ss

http://www.ttlsa.com/linux-command/ss-replace-netstat/

```shell
# 本地所有端口
ss -l
# 加上进程信息
ss -lp
# 所有tcp socket
ss -t -a
# 所有udp socket
ss -u -a
# 已建立的smtp连接
ss -o state established '(dport=:smtp or sport=:smtp)'
# 目标为某IP的tcp连接
ss -t -o state established dst 192.168.64.130
# 源头为某IP的tcp连接
ss -t -o state established src 192.168.64.129
# socket汇总
ss -s
```



#### IO相关

##### vmstat

见上面vmstat介绍





### 一些有用，裸机可能没有的工具

httpd-tools： 辅助Apache HTTP Server的软件包，提供ab压测

perf：针对某个进程的CPU分析，定位到方法调用

sysstat：提供sar、iostat、mpstat、pidstat等工具，还提供了定时统计

lsof：lists openfiles，基本功能是列出系统打开的文件，常用于网络问题分析
