## apt

### 什么是apt

 apt（Advanced Packaging Tool）基于 DEB 包管理，是一个常见于 Debian 和 Ubuntu 中的软件包管理器，也用于 Kali 等系统中

对应着dpkg

### apt和apt-get的关系

虽然有些微小的差异，但也可以认为：`apt` 是 `apt-get` 的超集，它包含 `apt-get`、`apt-cache` 和 `apt-config` 中最常用命令选项的集合。

一般的应用场景下 `apt` 和 `apt-get` 可以互用

不过也确实可以作一些区分：

> WARNING : apt does not have a stable CLI interface.

编写高可靠需求的自动化脚本时，使用apt-get；其余时候可以使用更简练的apt命令

### 常用命令
|command|description|
|---|---|
|sudo apt update|列出所有可更新的软件清单命令|
|sudo apt upgrade|升级软件包|
|apt list --upgradeable|列出可更新的软件包及版本信息|
|sudo apt full-upgrade|升级软件包，升级前先删除需要更新软件包|
|sudo apt install <package_name>|安装指定的软件命令|
|sudo apt install <package_1> <package_2> <package_3>|安装多个软件包|
|sudo apt update <package_name>|更新指定的软件命令|
|sudo apt show <package_name>|显示软件包具体信息,例如：版本号，安装大小，依赖关系等等|
|sudo apt remove <package_name>|删除软件包命令|
|sudo apt autoremove|清理不再使用的依赖和库文件|
|sudo apt purge <package_name>|移除软件包及配置文件|
|sudo apt search \<keyword\>|从软件源中查找软件包|
|**apt list --installed**|**列出所有已安装的包**|
|apt list --all-versions|列出所有已安装的包的版本信息|
|apt help|帮助|

- `apt-get update`：是同步`/etc/apt/sources.list`和`/etc/apt/sources.list.d`中列出的软件源的软件包版本，这样才能获取到最新的软件包。
- `apt-get upgrade`：是更新已安装的所有或者指定软件包，升级之到本地索引中的对应版本。因此，在执行 `upgrade` 之前一般要执行`update`，这样安装的才是最新的版本。

### 编辑source.list文件

以 ubuntu20.04lts 为例

在使用`apt`时，我们需要维护一个存储着源地址信息的文本文件：`/etc/apt/source.list`，

`apt`可以通过这些地址来查询软件最新版本并通过他们进行更新

source.list内容(已去除部分注释)：


```bash
deb http://archive.ubuntu.com/ubuntu/ focal main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal main restricted
deb http://archive.ubuntu.com/ubuntu/ focal-updates main restricted
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates main restricted
deb http://archive.ubuntu.com/ubuntu/ focal universe
# deb-src http://archive.ubuntu.com/ubuntu/ focal universe
deb http://archive.ubuntu.com/ubuntu/ focal-updates universe
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates universe
deb http://archive.ubuntu.com/ubuntu/ focal multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal multiverse
deb http://archive.ubuntu.com/ubuntu/ focal-updates multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal-updates multiverse
deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu/ focal-security main restricted
# deb-src http://security.ubuntu.com/ubuntu/ focal-security main restricted
deb http://security.ubuntu.com/ubuntu/ focal-security universe
# deb-src http://security.ubuntu.com/ubuntu/ focal-security universe
deb http://security.ubuntu.com/ubuntu/ focal-security multiverse
# deb-src http://security.ubuntu.com/ubuntu/ focal-security multiverse
```

**解析规则：**

总的来说，解析list时遵循以下规则：

**uri + "dists" + ubuntu版本信息 + 索引分类 + 仓库类型**

1. 仓库类型：

deb： 二进制包仓库

deb-src： 二进制包的源码库

2. uri：

URI：库所在的地址，可以是网络地址，也可以是本地的镜像地址

3. 版本信息：

就是当前Ubuntu对应的版本代号。可以用命令lsb_release -sc来查看当前系统的代号。

20.04lts的代号是 `focal`，所以所有的uri后都会有 `focal`

具体则有五种后缀：

- 无后缀 - 一般不考虑的随发布的source

- Security - Important Security Updates.

- Updates - Recommended Updates.

- Proposed - Pre-released Updates.

- Backports - Unsupported Updates.

4. 索引分类：

components： 软件的性质（free或non-free等）

共有四种：

- main: 完全的自由软件。

- restricted: 不完全的自由软件。

- universe: Ubuntu官方不提供支持与补丁，全靠社区支持。

- multiverse：非自由软件，完全不提供支持和补丁。

例如，现有一个源配置如下：

> deb http://archive.ubuntu.com/ubuntu/ focal main restricted

那么，解析出的结果为：

http://cn.archive.ubuntu.com/ubuntu/dists/focal/main

http://cn.archive.ubuntu.com/ubuntu/dists/focal/restricted

![image-20220128141631953](C:\Users\m1518\OneDrive\文档\Document\Linux\compareAPTandYUM01.png)

![image-20220128141726663](C:\Users\m1518\OneDrive\文档\Document\Linux\compareAPTandYUM02.png)

`deb-src `会对应 `source`，`deb` 则会对应 `binary-xxx`，`xxx` 就是 `arch`，比如 `i386` (32位)或是 `amd64` (64位)。

如需指定 arch，则对应：

> deb [arch=amd64] http://cn.archive.ubuntu.com/ubuntu/ focal main

会指向：

> http://cn.archive.ubuntu.com/ubuntu/dists/focal/main/binary-amd64/

### 编辑 /etc/apt/sources.list.d/ 目录 

和`sources.list`功能一样的是`/etc/apt/sources.list.d`目录 

在此目录下，我们可以随意定制我们所指定的软件源，只要以 `.list` 结尾即可

`sources.list.d` 目录下的 `.list` 文件为软件源的管理提供了全新的思路，我们亦可以用此来安装第三方的软件。

示例：

用 `/etc/apt/sources.list.d/google-chrome.list` 文件来暂存 `google chrome` 的源

```bash
>> cat google-chrome.list
deb http://dl.google.com/linux/chrome/deb/ stable main
```

 

## yum

### 介绍：

yum(Yellow dog Updater, Modified)基于 RPM 包管理，是一个应用在 Fedora 和 RedHat 系Linux中的软件包管理器。

对应着rpm

### 常用命令：

|command|description|
|:--|:--|
|yum search|使用 yum (在源内)查找软件包 |
|yum install \<package_name\>|仅安装指定的软件命令|
|yum update \<package_name\>|仅更新指定的软件命令|
|yum remove \<package_name\>|删除软件包命令|
|yum list|列出(源内)所有可安装的软件包 |
|yum list updates |列出所有可更新的软件包 |
|**yum list installed** |**列出所有已安装的软件包**|
|yum list extras |列出所有已安装但不在 Yum Repository 内的软件包 |
|yum info |使用 YUM 获取软件包信息 |
|yum info updates |列出所有可更新的软件包信息 |
|**yum info installed** |**列出所有已安装的软件包信息** |
|yum info extras |列出所有已安装但不在 Yum Repository 内的软件包信息 |
|yum provides|列出软件包提供哪些文件 |

### 修改yum源：

1. 进入yum源配置目录： 
```bash
cd /etc/yum.repos.d
```

2. 备份原配置文件：

```bash
sudo mv CentOS-Base.repo CentOS-Base.repo.backup
```

3. 下载新配置文件：

```bash
sudo wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# http://mirrors.aliyun.com/repo先找到对应系统，这里是centos7
# 没有wget就用curl，一定要用就在mv前线下载wget
```

4. 清除缓存

```bash
sudo yum clean all
sudo yum makecache
```

5. 查看是否更新成功

```bash
yum repolist
```

6. 更新所有软件

```bash
yum update -y 
```

当然，我们也可以使用yum自带的插件——fastest-mirror

![image-20220128150420505](C:\Users\m1518\OneDrive\文档\Document\Linux\compareAPTandYUM03.png)

> The fastest mirror plugin is designed for use in repository configurations where you have more than 1 mirror in a repo configuration.
>
> After fastestmirror is installed, make sure that it is enabled. 
>
> Edit the file `/etc/yum/pluginconf.d/fastestmirror.conf` and ensure that it contains the following lines:
>
> ```bash
> [main]
> verbose = 0
> socket_timeout = 3
> enabled = 1
> hostfilepath = /var/cache/yum/timedhosts.txt
> maxhostfileage = 1
> ```
>
> To exclude a specific mirror, TLD, or something in between, add an 'exclude=' line to `/etc/yum/pluginconf.d/fastestmirror.conf`:
>
> ```bash
> [main]
> ...
> exclude=.gov, facebook, myspace, junk-mirror.com
> ```
>
> [PackageManagement/Yum/FastestMirror - CentOS Wiki](https://wiki.centos.org/PackageManagement/Yum/FastestMirror#:~:text=fastestmirror The fastest mirror plugin is designed for,by fastest to slowest for use by yum.)



### dnf——新一代的RPM软件包管理器：

DNF 包管理器作为 YUM 包管理器的升级替代品，它能自动完成更多的操作。

[DNF/zh-cn - Fedora Project Wiki](https://fedoraproject.org/wiki/DNF/zh-cn)

## 一些对比

| 对比项 | rpm  | yum  | dpkg | apt  |
| ------ | ---- | ---- | ---- | ---- |
|系列|	RedHat系|	RedHat系|	Debian系|	Debian系|
|区别	|包安装工具|	依赖管理工具|	包安装工具|	依赖管理工具|
|**查询已安装**	|**rpm -qa**| **yum list installed** |	**dkpg -l**	|**apt list –installed**|
|安装| rpm -i package.rpm <br />或<br /> rpm –ivh http://www.xxx.net/package.rpm |	yum install	|dpkg -i package.deb| apt install package |
|更新|	rpm –U software.rpm|	yum update	|	|apt upgrade|
|移除软件包|	rpm -e [module1][module2]…|	yum remove	|dpkg -r package|apt remove package|
|移除软件包及配置		|||	dpkg -P|	apt purge package|
|下载的包存放位置		||||		/var/cache/apt/archives|
|软件安装默认位置	|rpm -ql	|||		/usr/share|
|可执行文件位置	|/usr/bin	|||		/usr/bin|
|配置文件位置	|/etc		|||	/etc|
|lib文件位置	|/usr/lib	||||
|使用手册	|/usr/share/doc	||||
|帮助文档|	/usr/share/man		||||
|更新				|||||

[yum与apt的区别_qq_26182553的博客-CSDN博客_yum和apt](https://blog.csdn.net/qq_26182553/article/details/79869666)

## RPM介绍

[RPM简介与基本使用 - 大师兄啊哈 - 博客园 (cnblogs.com)](https://www.cnblogs.com/harrymore/p/8665154.html#_label3_0)
