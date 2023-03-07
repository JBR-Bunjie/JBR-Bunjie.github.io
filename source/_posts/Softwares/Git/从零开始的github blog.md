## 为什么使用Github作为博客的托管网站？

## 怎么搭建你的博客？

首先你需要一个github账号，并在其中创建一个全新的公开仓库

### 创建公开仓库 xxx.github.io



### 为你的github账号添加ssh keys

#### 什么是ssh keys？

#### 为什么需要ssh keys

因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的

这也是GitHub上的项目中，你可以准许或禁止某些人push新内容的原因

#### 怎么获取ssh keys

如果你已经完成了本地git global user.name和user.email的设置的话

你可以直接使用如下命令生成你的ssh keys：

> ssh-keygen  #可选： -t rsa -C "你的邮箱地址"
>
> > 在此过程中，你可能会遇到要你选择保留公钥和私钥，设置密码的选项，直接回车就可以了，当然你也可以根据自己需要做出选择

win下生成的ssh keys会保存在C:\Users\当前用户\lzw.ssh中，用记事本打开即可查看

`请不要将你的ssh keys密钥告诉别人`

linux下生成的SSH key文件保存在中～/.ssh/id_rsa.pub

#### 怎么添加ssh keys

1. 进入Github Settings SSH and GPG keys页面[SSH and GPG keys (github.com)](https://github.com/settings/keys)

2. 选择New SSH Key
3. 自定义title，并将你的id_rsa.pub的全部内容复制到Key中，确定即可

**添加完ssh keys后，就可以在这台电脑上连接你的GitHub仓库了**









