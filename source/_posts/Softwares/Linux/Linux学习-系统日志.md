## 清空系统历史命令

### 1. history -c

该命令只清空本次登入的所有输出命令，且不清空.bash_history文件

所以下次登陆后，旧命令还将出现，历史命令是存在于当前用户根目录下的./bash_history文件。

### 2. echo > $HOME/.bash_history

每个用户根目录下都有一个.bash_history文件用于保存历史命令，当每次注销时，本次登陆所执行的命令将被写入该文件。所以可以清空该文件，下次登陆后上次保存的命令将消失，清空效果将在下次登陆生效。

### 3. 利用设备黑洞

对history文件执行文本清空

```bash
cat /dev/null > /root/.bash_history
```

清空文件内容：

```bash
cat /dev/null > [yourfilename]
```

