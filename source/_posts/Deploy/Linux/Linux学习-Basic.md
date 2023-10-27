---
title: Linux Basic
date: 2022-1-16 11:11:11
tags:
  - Linux
categories:
  - Linux
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Linux Basic

## apt 配置

### apt 切换国内源

linux 系统默认使用的软件源都是国外源，国内访问速度过慢，所以改为国内镜像源

1. `sudo su`进入 root 模式
2. `vim /etc/apt/sources.list`编辑软件源配置文件
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/53be24e5e1d54876a5808193d0095674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dTQTE2MzU=,size_16,color_FFFFFF,t_70)
3. 按 i 进入 vim 的编辑模式，用`#`将 deb 一行的内容注释掉，然后换成国内源地址,这里我直接用阿里云源

```bash
官方源
deb http://http.kali.org/kali kali-rolling main non-free contrib
deb-src http://http.kali.org/kali kali-rolling main non-free contrib
中科大源
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib
阿里云源
deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free contrib
清华大学源
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free
浙大源
deb http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
deb-src http://mirrors.zju.edu.cn/kali kali-rolling main contrib non-free
东软大学源
deb http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib
deb-src http://mirrors.neusoft.edu.cn/kali kali-rolling/main non-free contrib
新加坡kali源
deb http://mirror.nus.edu.sg/kali/kali/ kali main non-free contrib
deb-src http://mirror.nus.edu.sg/kali/kali/ kali main non-free contrib
163 Kali源
deb http://mirrors.163.com/debian wheezy main non-free contrib
deb-src http://mirrors.163.com/debian wheezy main non-free contrib
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5f161f66a114224a54f15ef3d82e0f8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dTQTE2MzU=,size_16,color_FFFFFF,t_70)
之后 Esc，然后 :wq 保存退出即可
\4. `apt-get update`更新索引
\5. `apt-get upgrade`更新软件



## 用户

In Linux, there are three types of owners: `user`, `group`, and `others` .

### Linux User

A user is the default owner and creator of the file. So this user is called owner as well.

> user == owner

### Linux Group

A user-group is a collection of users. Users that belonging to a group will have the same Linux group permissions to access a file/ folder.

You can use groups to assign permissions in a bulk instead of assigning them individually. A user can belong to more than one group as well.

### Other

Any users that are not part of the user or group classes belong to this class.

## 更换用户

### 创建一个全新的用户账号：

**useradd** + **passwd**

### 更换：

**su 命令**：

```bash
su root
su - root # 不一样！
# "su r"只是切换了用户，要想连shell环境一起切换就用后边的"su - root"。
```

## 文件

### 文件分类

Linux 共有七类文件

#### 普通文件类型 [-]:

Linux 中最多的一种文件类型, 包括 纯文本文件(ASCII)；二进制文件(binary)；数据格式的文件(data);各种压缩文件。其第一个属性为 [-]

#### 目录文件 [d]:

就是目录， 能用 # cd 命令进入的。第一个属性为 [d]，例如 [drwxrwxrwx]

#### 块设备文件 [b]:

块设备文件就是存储数据以供系统存取的接口设备，简单而言就是硬盘。

例如一号硬盘的代码是 /dev/hda1 等文件。第一个属性为 [b]

#### 字符设备 [c]:

字符设备文件即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]

#### 套接字文件 [s]:

这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。

第一个属性为 [s]，最常在 /var/run 目录中看到这种文件类型

#### 管道文件 [p]:

FIFO 也是一种特殊的文件类型，它主要的目的是，解决多个程序同时存取一个文件所造成的错误。

FIFO 是 first-in-first-out(先进先出)的缩写。第一个属性为 [p]

#### 链接文件 [l]:

类似 Windows 下面的快捷方式。第一个属性为 [l]，例如 [lrwxrwxrwx]

## How to Change File Permissions and Ownership in Linux

### chmod 与 chown 的区别

**chmod** - Used for Changing Permissions 用于改变 具体文件或目录 之于某个用户或用户组的 权限关系: permission

**chown** - Used for Changing Ownership 用与改变 具体文件或目录 之于某个用户或用户组的 归属关系: ownership

### CHMOD PART

#### Syntax

```bash
chmod [-options] [permissions] [filename]
```

- `permissions` can be **read**, **write**, **execute** or a **combination** of them.
- `filename` is the name of the file for which the permissions need to change. This parameter can also be a list if files to change permissions in bulk.

We can change permissions using two modes:

1. **Symbolic mode**: this method uses symbols like `u`, `g`, `o` to represent users, groups, and others. Permissions are represented as `r, w, x` for read write and execute, respectively. You can modify permissions using +, - and =.
2. **Absolute mode**: this method represents permissions as 3-digit octal numbers ranging from 0-7.

#### Example

```bash
chmod u+x mymotd.sh
# To add execution rights (x) to user(or-> the file owner)(u) using symbolic mode, we can use the command above;

chmod 777 test.txt
# 把三个分区看作三段被拼接的二进制
# 111 -> 7, rwx, u, g, o;
```

### CHOWN PART

#### Syntax & Example

```bash
chown [-options] [user:group] [filename]/[path]

chown -R root /root/test # 改变文件夹的归属
chown :admins /opt/script # To change group ownership, we can use chown by preceding the group name by a colon ':'
```

[Linux 中 chown 和 chmod 的区别和用法（转） - EasonJim - 博客园 (cnblogs.com)](https://www.cnblogs.com/EasonJim/p/6525242.html)

[Linux chmod and chown – How to Change File Permissions and Ownership in Linux (freecodecamp.org)](https://www.freecodecamp.org/news/linux-chmod-chown-change-file-permissions/)



