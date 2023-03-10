---
title: 直接利用命令行传输文件
date: 2022-1-15 11:07:03
tags:
  - FTP
  - SCP
categories:
  - Linux
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## FTP命令

```bash
ftp ftp.kernel.org #发起链接请求 
```

**参数**：

- -d 详细显示指令执行过程，便于排错或分析程序执行的情形。
- -i 关闭互动模式，不询问任何问题。
- -g 关闭本地主机文件名称支持特殊字符的扩充特性。
- -n 不使用自动登陆。
- -v 显示指令执行过程。

几个简单的命令：

```bash
ls # 显示全部内容

get <filename> # 下载
mget <filename> # 批量下载

put <filename> # 上传
mput <filename> # 批量上传

cdup # 快速移动到当前目录的父目录

delete # 删除
mdelete # p
```

详细可以看->[How to Use the FTP Command on Linux (howtogeek.com)](https://www.howtogeek.com/412626/how-to-use-the-ftp-command-on-linux/)

顺便，关于sftp...

The FTP commands we have described above will work just the same in an SFTP session, with the following exceptions.

- To delete a file use `rm` (FTP uses `delete`)
- To delete multiple files use `rm` (FTP uses `mdelete`)
- To move to the parent directory use `cd ..` (FTP uses `cdup`)

---

## scp命令

推荐：[Linux scp command help and examples (computerhope.com)](https://www.computerhope.com/unix/scp.htm)

scp命令(secure copy)用于 Linux 之间进行的文件和目录复制。

scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令

scp 是加密的，[rcp](https://www.runoob.com/linux/linux-comm-rcp.html) 是不加密的，scp 是 rcp 的加强版

### 语法

```bash
# scp [-1246BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file] [-l limit] [-o ssh_option] [-P port] [-S program][[user@]host1:]file1 [...] [[user@]host2:]file2
```

简易写法:

```bash
#scp [可选参数] file_source file_target 
```

**参数说明：**

- -1： 强制scp命令使用协议ssh1
- -2： 强制scp命令使用协议ssh2
- -4： 强制scp命令只使用IPv4寻址
- -6： 强制scp命令只使用IPv6寻址
- -B： 使用批处理模式（传输过程中不询问传输口令或短语）
- -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
- -p：保留原文件的修改时间，访问时间和访问权限。
- -q： 不显示传输进度条。
- -r： 递归复制整个目录。
- -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
- -c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
- -F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
- -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
- -l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
- -o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
- -P port：注意是大写的P, port是指定数据传输用到的端口号
- -S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

### 实例

#### 1、从本地复制到远程

命令格式：

```bash
scp local_file remote_username@remote_ip:remote_folder 
#或者 
scp local_file remote_username@remote_ip:remote_file 
#或者 
scp local_file remote_ip:remote_folder 
#或者 
scp local_file remote_ip:remote_file 
```

解释：

- 第1,2个指定了用户名，命令执行后需要再输入密码，第1个仅指定了远程的目录，文件名字不变，第2个指定了文件名；
- 第3,4个没有指定用户名，命令执行后需要输入用户名和密码，第3个仅指定了远程的目录，文件名字不变，第4个指定了文件名；

应用实例：

```bash
scp /home/space/music/1.mp3 root@www.runoob.com:/home/root/others/music 
scp /home/space/music/1.mp3 root@www.runoob.com:/home/root/others/music/001.mp3 
scp /home/space/music/1.mp3 www.runoob.com:/home/root/others/music 
scp /home/space/music/1.mp3 www.runoob.com:/home/root/others/music/001.mp3 
```

复制目录命令格式：

```bash
scp -r local_folder remote_username@remote_ip:remote_folder 
#或者 
scp -r local_folder remote_ip:remote_folder 
```

- 第1个指定了用户名，命令执行后需要再输入密码；
- 第2个没有指定用户名，命令执行后需要输入用户名和密码；

应用实例：

```bash
scp -r /home/space/music/ root@www.runoob.com:/home/root/others/ 
scp -r /home/space/music/ www.runoob.com:/home/root/others/ 
```

上面命令将本地 music 目录复制到远程 others 目录下。

#### 2、从远程复制到本地

**从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可**，如下实例

应用实例：

```bash
scp root@www.runoob.com:/home/root/others/music /home/space/music/1.mp3 
scp -r www.runoob.com:/home/root/others/ /home/space/music/
```

### 说明

1.如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号，命令格式如下：

```bash
#scp 命令使用端口号 4588
scp -P 4588 remote@www.runoob.com:/usr/local/sin.sh /home/administrator
```

2.使用scp命令要确保使用的用户具有可读取远程服务器相应文件的权限，否则scp命令是无法起作用的。

---

## curl命令

official site：[curl](https://curl.se/)

source code：[curl/curl: A command line tool and library for transferring data with URL syntax, supporting DICT, FILE, FTP, FTPS, GOPHER, GOPHERS, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, MQTT, POP3, POP3S, RTMP, RTMPS, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET and TFTP. libcurl offers a myriad of powerful features (github.com)](https://github.com/curl/curl)

official tutorial：[curl - Tutorial](https://curl.se/docs/manual.html)

third party tutorial：

[curl 的用法指南 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)

[Curl Cookbook (catonmat.net)](https://catonmat.net/cookbooks/curl)

不带有任何参数时，curl 就是发出 GET 请求。

```bash
curl https://www.example.com
```

上面命令向`www.example.com`发出 GET 请求，服务器返回的内容会在命令行输出。

### -A

`-A`参数指定客户端的用户代理标头，即`User-Agent`。curl 的默认用户代理字符串是`curl/[version]`。

```bash
curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
```

上面命令将`User-Agent`改成 Chrome 浏览器。

```bash
curl -A '' https://google.com
```

上面命令会移除`User-Agent`标头。

也可以通过`-H`参数直接指定标头，更改`User-Agent`。

```bash
curl -H 'User-Agent: php/1.0' https://google.com
```

### -b

`-b`参数用来向服务器发送 Cookie。

```bash
curl -b 'foo=bar' https://google.com
```

上面命令会生成一个标头`Cookie: foo=bar`，向服务器发送一个名为`foo`、值为`bar`的 Cookie。

```bash
curl -b 'foo1=bar;foo2=bar2' https://google.com
```

上面命令发送两个 Cookie。

```bash
curl -b cookies.txt https://www.google.com
```

上面命令读取本地文件`cookies.txt`，里面是服务器设置的 Cookie（参见`-c`参数），将其发送到服务器。

### -c

`-c`参数将服务器设置的 Cookie 写入一个文件。

```bash
curl -c cookies.txt https://www.google.com
```

上面命令将服务器的 HTTP 回应所设置 Cookie 写入文本文件`cookies.txt`。

### -d

`-d`参数用于发送 POST 请求的数据体。

```bash
curl -d'login=emma＆password=123'-X POST https://google.com/login
# 或者
curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
```

使用`-d`参数以后，HTTP 请求会自动加上标头`Content-Type : application/x-www-form-urlencoded`。并且会自动将请求转为 POST 方法，因此可以省略`-X POST`。

`-d`参数可以读取本地文本文件的数据，向服务器发送。

```bash
curl -d '@data.txt' https://google.com/login
```

上面命令读取`data.txt`文件的内容，作为数据体向服务器发送。

### --data-urlencode

`--data-urlencode`参数等同于`-d`，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码。

```bash
curl --data-urlencode 'comment=hello world' https://google.com/login
```

上面代码中，发送的数据`hello world`之间有一个空格，需要进行 URL 编码。

### -e

`-e`参数用来设置 HTTP 的标头`Referer`，表示请求的来源。

```bash
curl -e 'https://google.com?q=example' https://www.example.com
```

上面命令将`Referer`标头设为`https://google.com?q=example`。

`-H`参数可以通过直接添加标头`Referer`，达到同样效果。

```bash
curl -H 'Referer: https://google.com?q=example' https://www.example.com
```

### -F

`-F`参数用来向服务器上传二进制文件。

```bash
curl -F 'file=@photo.png' https://google.com/profile
```

上面命令会给 HTTP 请求加上标头`Content-Type: multipart/form-data`，然后将文件`photo.png`作为`file`字段上传。

`-F`参数可以指定 MIME 类型。

```bash
curl -F 'file=@photo.png;type=image/png' https://google.com/profile
```

上面命令指定 MIME 类型为`image/png`，否则 curl 会把 MIME 类型设为`application/octet-stream`。

`-F`参数也可以指定文件名。

```bash
curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
```

上面命令中，原始文件名为`photo.png`，但是服务器接收到的文件名为`me.png`。

### -G

`-G`参数用来构造 URL 的查询字符串。

```bash
curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
```

上面命令会发出一个 GET 请求，实际请求的 URL 为`https://google.com/search?q=kitties&count=20`。如果省略`--G`，会发出一个 POST 请求。

如果数据需要 URL 编码，可以结合`--data--urlencode`参数。

```bash
curl -G --data-urlencode 'comment=hello world' https://www.example.com
```

### -H

`-H`参数添加 HTTP 请求的标头。

```bash
curl -H 'Accept-Language: en-US' https://google.com
```

上面命令添加 HTTP 标头`Accept-Language: en-US`。

```bash
curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
```

上面命令添加两个 HTTP 标头。

```bash
curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
```

上面命令添加 HTTP 请求的标头是`Content-Type: application/json`，然后用`-d`参数发送 JSON 数据。

### -i

`-i`参数打印出服务器回应的 HTTP 标头。

```bash
curl -i https://www.example.com
```

上面命令收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码。

### -I

`-I`参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。

```bash
curl -I https://www.example.com
```

上面命令输出服务器对 HEAD 请求的回应。

`--head`参数等同于`-I`。

```bash
curl --head https://www.example.com
```

### -k

`-k`参数指定跳过 SSL 检测。

```bash
curl -k https://www.example.com
```

上面命令不会检查服务器的 SSL 证书是否正确。

### -L

`-L`参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

```bash
curl -L -d 'tweet=hi' https://api.twitter.com/tweet
```

### --limit-rate

`--limit-rate`用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。

```bash
curl --limit-rate 200k https://google.com
```

上面命令将带宽限制在每秒 200K 字节。

### -o

`-o`参数将服务器的回应保存成文件，等同于`wget`命令。

```bash
curl -o example.html https://www.example.com
```

上面命令将`www.example.com`保存成`example.html`。

### -O

`-O`参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。

```bash
curl -O https://www.example.com/foo/bar.html
```

上面命令将服务器回应保存成文件，文件名为`bar.html`。

### -s

`-s`参数将不输出错误和进度信息。

```bash
curl -s https://www.example.com
```

上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。

如果想让 curl 不产生任何输出，可以使用下面的命令。

```bash
curl -s -o /dev/null https://google.com
```

### -S

`-S`参数指定只输出错误信息，通常与`-s`一起使用。

```bash
curl -s -o /dev/null https://google.com
```

上面命令没有任何输出，除非发生错误。

### -u

`-u`参数用来设置服务器认证的用户名和密码。

```bash
curl -u 'bob:12345' https://google.com/login
```

上面命令设置用户名为`bob`，密码为`12345`，然后将其转为 HTTP 标头`Authorization: Basic Ym9iOjEyMzQ1`。

curl 能够识别 URL 里面的用 户名和密码。

```bash
curl https://bob:12345@google.com/login
```

上面命令能够识别 URL 里面的用户名和密码，将其转为上个例子里面的 HTTP 标头。

```bash
curl -u 'bob' https://google.com/login
```

上面命令只设置了用户名，执行后，curl 会提示用户输入密码。

### -v

`-v`参数输出通信的整个过程，用于调试。

```bash
curl -v https://www.example.com
```

`--trace`参数也可以用于调试，还会输出原始的二进制数据。

```bash
curl --trace - https://www.example.com
```

### -x

`-x`参数指定 HTTP 请求的代理。

```bash
curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
```

上面命令指定 HTTP 请求通过`myproxy.com:8080`的 socks5 代理发出。

如果没有指定代理协议，默认为 HTTP。

```bash
curl -x james:cats@myproxy.com:8080 https://www.example.com
```

上面命令中，请求的代理使用 HTTP 协议。

### -X

`-X`参数指定 HTTP 请求的方法。

```bash
curl -X POST https://www.example.com
```

上面命令对`https://www.example.com`发出 POST 请求。

---

## wget命令

[Wget - GNU Project - Free Software Foundation](https://www.gnu.org/software/wget/)

