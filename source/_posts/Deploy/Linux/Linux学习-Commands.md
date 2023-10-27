---
title: Linux Commands
date: 2022-1-16 11:07:03
tags:
  - Linux
categories:
  - Linux
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Linux Commands

## 清空系统历史命令

### 1. history -c

该命令只清空本次登入的所有输出命令，且不清空.bash_history 文件

所以下次登陆后，旧命令还将出现，历史命令是存在于当前用户根目录下的./bash_history 文件。

### 2. echo > $HOME/.bash_history

每个用户根目录下都有一个.bash_history 文件用于保存历史命令，当每次注销时，本次登陆所执行的命令将被写入该文件。所以可以清空该文件，下次登陆后上次保存的命令将消失，清空效果将在下次登陆生效。

### 3. 利用设备黑洞

对 history 文件执行文本清空

```bash
cat /dev/null > /root/.bash_history
```

清空文件内容：

```bash
cat /dev/null > [yourfilename]
```



## 检查计算机状态

### 硬件信息

```bash
> cat /etc/os-rel
> cat /proc/cpuinfo # 查看CPU信息 
> cat /proc/cpuinfo | grep "model name" # 仅查看CPU内核

> cat /proc/meminfo # 查看内存
> cat /proc/meminfo | grep MemTotal # 仅查看内存大小

> fdisk -l # 硬盘大小 
> fdisk -l | grep Disk # 仅查看Disk信息

> lsusb -tv # 列出所有USB设备的linux系统信息命令
> lspci -tv # 列出所有PCI设备
```

### 了解计算机当前操作系统相关信息

```bash
> uname -a # 查看内核/操作系统/CPU信息 
> hostname # 查看计算机名称
> cat /etc/issue # 查看操作系统版本，是数字1不是字母L
> env # 查看环境变量资源
> df -h # 查看各分区使用情况
```

### 检测当前计算机进程

```bash
> top # 实时显示: 进程PID 用户 CPU占用率 等等信息
> ps -ef # 查看所有进程
> free -m # 查看内存使用量和交换区使用量
> lsmod # 列出加载的内核模块
```

### 防火墙

```bash
> firewall-cmd --state # 查看火墙状态
```

### 环境变量

```bash
> env # 查看环境变量资源
```


head 命令可用于查看文件的开头部分的内容，有一个常用的参数 **-n** 用于显示行数，默认为 10，即显示 10 行的内容。

### nohup：[Linux nohup 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-nohup.html)

```bash
nohup Command [ Arg … ] [　& ]
```

**Command**：要执行的命令。

**Arg**：一些参数，可以指定输出文件。

**&**：让命令在后台执行，终端退出后命令仍旧执行。


### 了解

### tar命令

tar命令语句范式：

```shell
tar [options][archive-file] [file or dir to be archived] #
```

### touch命令：

Linux touch命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

ls -l 可以显示档案的时间记录。

###  语法

```
touch [-acfm][-d<日期时间>][-r<参考文件或目录>] [-t<日期时间>][--help][--version][文件或目录…]
```

- **参数说明**：
- a 改变档案的读取时间记录。
- m 改变档案的修改时间记录。
- c 假如目的档案不存在，不会建立新的档案。与 --no-create 的效果一样。
- f 不使用，是为了与其他 unix 系统的相容性而保留。
- r 使用参考档的时间记录，与 --file 的效果一样。
- d 设定时间与日期，可以使用各种不同的格式。
- t 设定档案的时间记录，格式与 date 指令相同。
- --no-create 不会建立新档案。
- --help 列出指令格式。
- --version 列出版本讯息。

### getsebool命令:

![image-20210918233125352](../../../images/Deploy/image-20210918233125352-16319790905121.png)

将enforcing修改为permissive或者disabled

### 查看命令路径：

```bash
type python3
# -------
where python3
# -------
whereis python3
```

### 启动服务

```bash
sudo service ssh start

#检测：
#ssh localhost
```

### 重启服务

```bash
sudo service sshd restart
```

## Reverse Area

```bash
# uname -a # 查看内核/操作系统/CPU信息 
# head -n 1 /etc/issue # 查看操作系统版本 
# cat /proc/cpuinfo # 查看CPU信息 
# hostname # 查看计算机名 
# lsmod # 列出加载的内核模块 
# env # 查看环境变量资源 
# free -m # 查看内存使用量和交换区使用量 
# df -h # 查看各分区使用情况 
# du -sh <目录名> # 查看指定目录的大小 
# grep MemTotal /proc/meminfo # 查看内存总量 
# grep MemFree /proc/meminfo # 查看空闲内存量 
# uptime # 查看系统运行时间、用户数、负载 
# cat /proc/loadavg # 查看系统负载磁盘和分区 
# mount | column -t # 查看挂接的分区状态 
# fdisk -l # 查看所有分区 
# swapon -s # 查看所有交换分区 
# hdparm -i /dev/hda # 查看磁盘参数(仅适用于IDE设备) 
# dmesg | grep IDE # 查看启动时IDE设备检测状况网络 
# ifconfig # 查看所有网络接口的属性 
# iptables -L # 查看防火墙设置 
# route -n # 查看路由表 
# netstat -lntp # 查看所有监听端口 
# netstat -antp # 查看所有已经建立的连接 
# netstat -s # 查看网络统计信息进程 
# ps -ef # 查看所有进程 
# top # 实时显示进程状态用户 
# w # 查看活动用户 
# id <用户名> # 查看指定用户信息 
# last # 查看用户登录日志 
# cut -d: -f1 /etc/passwd # 查看系统所有用户 
# cut -d: -f1 /etc/group # 查看系统所有组 
# crontab -l # 查看当前用户的计划任务服务 
# chkconfig –list # 列出所有系统服务 
# chkconfig –list | grep on # 列出所有启动的系统服务程序 
# rpm -qa # 查看所有安装的软件包
# du -sh # 查看指定目录的大小
# 十五、grep MemTotal /proc/meminfo # 查看内存总量
# 十六、grep MemFree /proc/meminfo # 查看空闲内存量
# 十七、uptime # 查看系统运行时间、用户数、负载
# 十八、cat /proc/loadavg # 查看系统负载磁盘和分区
# 十九、mount | column -t # 查看挂接的分区状态
# 二十、fdisk -l # 查看所有分区
# 二十一、swapon -s # 查看所有交换分区
# 二十二、hdparm -i /dev/hda # 查看磁盘参数(仅适用于IDE设备)
# 二十三、dmesg | grep IDE # 查看启动时IDE设备检测状况网络
# 二十四、ifconfig # 查看所有网络接口的属性
# 二十五、iptables -L # 查看防火墙设置
# 二十六、route -n # 查看路由表
# 二十七、netstat -lntp # 查看所有监听端口
# 二十八、netstat -antp # 查看所有已经建立的连接
# 二十九、netstat -s # 查看网络统计信息进程
# 三十、ps -ef # 查看所有进程
# 三十一、top # 实时显示进程状态用户
# 三十二、w # 查看活动用户
# 三十三、id # 查看指定用户信息
# 三十四、last # 查看用户登录日志
# 三十五、cut -d: -f1 /etc/passwd # 查看系统所有用户
# 三十六、cut -d: -f1 /etc/group # 查看系统所有组
# 三十七、crontab -l # 查看当前用户的计划任务服务
# 三十七、chkconfig –list # 列出所有系统服务
# 三十八、chkconfig –list | grep on # 列出所有启动的系统服务程序
# 三十九、rpm -qa # 查看所有安装的软件包
# 四十、cat /proc/cpuinfo ：查看CPU相关参数的linux系统命令
# 四十一、cat /proc/partitions ：查看linux硬盘和分区信息的系统信息命令
# 四十二、cat /proc/meminfo ：查看linux系统内存信息的linux系统命令
# 四十三、cat /proc/version ：查看版本，类似uname -r
# 四十四、cat /proc/ioports ：查看设备io端口
# 四十五、cat /proc/interrupts ：查看中断
# 四十六、cat /proc/pci ：查看pci设备的信息
# 四十七、cat /proc/swaps ：查看所有swap分区的信息
```

