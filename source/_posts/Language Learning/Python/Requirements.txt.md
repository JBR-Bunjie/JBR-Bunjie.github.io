---
title: requirements.txt in Python project
date: 2022-12-23 12:23:23
tags:
  - Python
  - Language Learning
categories:
  - Python
  - Language Learning
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# requirements.txt in Python project

写 python 程序的时候，我们经常的会下载很多外部模块，当我们编写完成后，准备在其他设备上部署的时候，那么新设备上需要安装我当前环境下的所有包——非常麻烦

我们可以利用 pip 来生成一个 requirements.txt 的文件，在新环境中通过读取这个文件中的模块名称进行环境的安装

### 生成 requirements.txt

在项目根目录打开 cmd/powershell

执行

```powershell
pip freeze > requirements.txt
```

例如：

![img](https://upload-images.jianshu.io/upload_images/8904450-2318a0b11dbf2fb0.png)

这时候项目根目录就会多一个`requirements.txt`文件，里面会记录我们项目需要的所以模块信息。具体说明可见：[pip freeze - pip documentation v22.0.3 (pypa.io)](https://pip.pypa.io/en/stable/cli/pip_freeze/)

- 请注意区分当前 terminal 中的 pip 是否和项目所使用的 pip 所一致，terminal 生成的 txt 是根据系统变量中 pip 的所有依赖包来生成 txt 的，可能跟项目实际所使用的有所不同
- 如果在创建项目都是包全局继承就比较悲剧，但鉴于 requirements.txt 的格式较为简单，在了解后可以尝试**手动创建**

### 使用 requirements.txt

新环境中通过此文件可以直接安装模块（注:需要先切换到 `requirements.txt`的上级目录，也就是项目根目录）

在项目**根目录**下执行

```powershell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt
```

安装至虚拟环境中命令

进入到了虚拟环境中：切到虚拟环境目录的 Script 文件下

```powershell
pip install -r D:\odoo13\odoo\requirements.txt
```

两个备用镜像源：

阿里 https://mirrors.aliyun.com/pypi/simple

清华 https://pypi.tuna.tsinghua.edu.cn/simple
