---
title: WSA Install third-party apps!
date: 2022-12-23 12:23:24
tags:
  - Installation
  - Windows
  - WSA
  - Android
categories:
  - Delopy and Installation
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

```powershell
# 第 0 步：确保已正确将 adb 命令加入到系统的环境变量
# 执行下面的命令能看到 adb 版本号则表示 ok
# 如有错误，请检查环境变量是否配置正确
adb version

# 第 1 步：连接 WSA
adb connect 127.0.0.1:58526
# 其中 127.0.0.1:58526 是刚才在 WSA 设置项中看到的 IP

# 第 2 步：安装 APK
# 连接成功之后，就能用下面命令来安装 APK 了
adb install {你的APK文件完整路径}
# 注意 .apk 的路径最好无中文且无空格，否则需要用英文双引号包裹。
# 你可在资源管理器上右键点击 apk 文件选「复制文件地址」获取完整路径
adb install d:\download\apk\weixin.apk
#下面是例子：
adb install "d:\下载\异次元 iPlaySoft.com\qq.apk"

# 最后按下回车即可安装
# 安装完成后，在 Windows 开始菜单的“所有应用”里就能找到你安装的 Android 应用
```

