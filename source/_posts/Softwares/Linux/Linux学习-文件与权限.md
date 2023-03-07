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



**su命令**：

```bash
su root
su - root # 不一样！
# "su r"只是切换了用户，要想连shell环境一起切换就用后边的"su - root"。
```

## 文件

### 文件分类

Linux共有七类文件

#### 普通文件类型 [-]:

Linux中最多的一种文件类型, 包括 纯文本文件(ASCII)；二进制文件(binary)；数据格式的文件(data);各种压缩文件。其第一个属性为 [-]

#### 目录文件 [d]:

就是目录， 能用 # cd 命令进入的。第一个属性为 [d]，例如 [drwxrwxrwx]

#### 块设备文件  [b]:

块设备文件就是存储数据以供系统存取的接口设备，简单而言就是硬盘。

例如一号硬盘的代码是 /dev/hda1等文件。第一个属性为 [b]

#### 字符设备 [c]:

字符设备文件即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]

#### 套接字文件 [s]:

这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。

第一个属性为 [s]，最常在 /var/run目录中看到这种文件类型

#### 管道文件 [p]:

FIFO也是一种特殊的文件类型，它主要的目的是，解决多个程序同时存取一个文件所造成的错误。

FIFO是first-in-first-out(先进先出)的缩写。第一个属性为 [p]

#### 链接文件 [l]:

类似Windows下面的快捷方式。第一个属性为 [l]，例如 [lrwxrwxrwx]

## How to Change File Permissions and Ownership in Linux

### chmod 与 chown的区别

**chmod** - Used for Changing Permissions 用于改变 具体文件或目录 之于某个用户或用户组的 权限关系: permission

**chown** - Used for Changing Ownership 用与改变  具体文件或目录 之于某个用户或用户组的 归属关系: ownership

### CHMOD PART

#### Syntax

```bash
chmod [-options] [permissions] [filename]
```

- `permissions` can be **read**, **write**, **execute** or a **combination** of them.
- `filename` is the name of the file for which the permissions need to change. This parameter can also be a list if files to change permissions in bulk.

We can change permissions using two modes:

1. **Symbolic mode**: this method uses symbols like `u`, `g`, `o` to represent users, groups, and others. Permissions are represented as  `r, w, x` for read write and execute, respectively. You can modify permissions using +, - and =.
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







[Linux中chown和chmod的区别和用法（转） - EasonJim - 博客园 (cnblogs.com)](https://www.cnblogs.com/EasonJim/p/6525242.html)

[Linux chmod and chown – How to Change File Permissions and Ownership in Linux (freecodecamp.org)](https://www.freecodecamp.org/news/linux-chmod-chown-change-file-permissions/)

