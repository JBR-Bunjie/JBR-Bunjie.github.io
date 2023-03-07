### apt切换国内源

linux 系统默认使用的软件源都是国外源，国内访问速度过慢，所以改为国内镜像源

1. `sudo su`进入root 模式
2. `vim /etc/apt/sources.list`编辑软件源配置文件
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/53be24e5e1d54876a5808193d0095674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dTQTE2MzU=,size_16,color_FFFFFF,t_70)
3. 按 i 进入 vim 的编辑模式，用`#`将deb一行的内容注释掉，然后换成国内源地址,这里我直接用阿里云源

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