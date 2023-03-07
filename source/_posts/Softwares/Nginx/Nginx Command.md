### 1.停止Nginx服务方法

- 立即停止服务
  这种方法比较强硬，无论进程是否在工作，都直接停止进程。

```bash
  nginx -s stop
```

- 从容停止服务
  这种方法较stop相比就比较温和一些了，需要进程完成当前工作后再停止。

```bash
nginx -s quit
```

- systemctl 停止
  systemctl属于[Linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Linux)命令

```bash
  systemctl stop nginx.service
```

- killall 方法杀死进程
  直接杀死进程，在上面无效的情况下使用，态度强硬，简单粗暴！

```bash
  killall nginx
```

### 2.启动Nginx

nginx直接启动

```bash
nginx
```

systemctl命令启动

```bash
systemctl start nginx.service
```

### 3.查看启动后记录

```bash
ps aux | grep nginx
```

### 4.重启Nginx服务

```bash
systemctl restart nginx.service
```

### 5.重新载入配置文件

当有系统配置文件有修改，用此命令，建议不要停止再重启，以防报错！

```bash
nginx -s reload
```

### 6.查看端口号

```bash
netstat -tlnp
```