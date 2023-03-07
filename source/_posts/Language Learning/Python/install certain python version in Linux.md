

# install certain python version in Linux

## -1: for Ubuntu:

just: apt install python3.8

## Zero, try to download python3.8 through yum

```bash
yum list | grep python3
```

find no python38, so choose to setup python3.8 through make install

## First, download the python file

go to website: https://www.python.org/ftp/python to choose correct file

> example: https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz

```bash
wget https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz
```

## unzip, make and install

1. 在编译前先在/usr/local建一个文件夹python3（作为python的安装路径，以免覆盖老的版本）

```bash
mkdir /usr/local/python3
```

2. 解压：

```bash
tar -zxvf Python-3.8.10.tgz
```

3. 进行指定目录的编译安装

```bash
./configure --prefix=/usr/local/python3 # 设置编译安装路径

make # 编译

make install # 安装 
```



4. 创建软链接

```bash
ln -s /usr/local/python3/bin/python3  /usr/bin/python3.8

# 修改旧版本软连接：ln -s /usr/local/python3/bin/python3  /usr/bin/pythonv
```



## 检验安装

```bash
python3.8 -V
# Python 3.8.10
```





[centOS下升级python版本，详细步骤 - leon-zyl - 博客园 (cnblogs.com)](https://www.cnblogs.com/leon-zyl/p/8422699.html)