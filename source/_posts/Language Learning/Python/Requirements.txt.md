# requirements.txt in Python project

写python程序的时候，我们经常的会下载很多外部模块，当我们编写完成后，准备在其他设备上部署的时候，那么新设备上需要安装我当前环境下的所有包——非常麻烦

我们可以利用pip来生成一个requirements.txt的文件，在新环境中通过读取这个文件中的模块名称进行环境的安装

### 生成requirements.txt


在项目根目录打开cmd/powershell

执行

```powershell
pip freeze > requirements.txt
```

例如：

![img](https://upload-images.jianshu.io/upload_images/8904450-2318a0b11dbf2fb0.png)

这时候项目根目录就会多一个`requirements.txt`文件，里面会记录我们项目需要的所以模块信息。具体说明可见：[pip freeze - pip documentation v22.0.3 (pypa.io)](https://pip.pypa.io/en/stable/cli/pip_freeze/)

+ 请注意区分当前terminal中的pip是否和项目所使用的pip所一致，terminal生成的txt是根据系统变量中pip的所有依赖包来生成txt的，可能跟项目实际所使用的有所不同
+ 如果在创建项目都是包全局继承就比较悲剧，但鉴于requirements.txt的格式较为简单，在了解后可以尝试**手动创建**

### 使用requirements.txt

新环境中通过此文件可以直接安装模块（注:需要先切换到 `requirements.txt`的上级目录，也就是项目根目录）

在项目**根目录**下执行
```powershell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt
```
 

安装至虚拟环境中命令

进入到了虚拟环境中：切到虚拟环境目录的Script文件下

```powershell
pip install -r D:\odoo13\odoo\requirements.txt
```



两个备用镜像源：

阿里 https://mirrors.aliyun.com/pypi/simple

清华 https://pypi.tuna.tsinghua.edu.cn/simple