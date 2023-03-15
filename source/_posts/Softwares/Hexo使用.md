---
title: Hello World, Hello Hexo
date: 2021-09-7 22:9:5
tags:
  - 使用hexo来搭建你的博客！
categories:
  - 配置方法
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

> Last Change At 3/16/2023

# Hexo从开箱到弃用

## Hexo的基础配置

### 安装hexo-cli

```powshell 
npm install hexo-cli -g
```

### 完成hexo的部署：

```powershell
npm install hexo-cli -g
hexo init <name> # 注意：hexo必须要在空文件夹下完成初始化，如果没有<name>，则在当前目录下完成初始化，否则新建一个<name>文件夹初始化 
cd <name> # 根据上个命令初始化的文件夹使用
npm install # 安装依赖
hexo server # 启动hexo服务器，这时候便可以访问所有 source/ 下的所有文章页面了
```

### 创建新的页面：

```powershell
hexo new [layout] <name> # 创建一个新的post页面，会出现在source文件夹下，内涵一个index.md
# 我们可以直接对这个md文件进行编辑，hexo会自动将这个文件渲染为一个html页面
```

:::warning

创建新文件夹或者页面时请注意 `\_config.yml` 中 `# Directory` 下项目的配置

:::

### 再次运行hexo：

```powershell
hexo clean # 清除之前hexo的缓存
hexo g # g或generate 根据md文件生成html等文件
hexo s # s或server 启动hexo服务器
```

### 配置Hexo快速部署：

[One-Command Deployment | Hexo](https://hexo.io/docs/one-command-deployment)

以git为例：

#### 安装hexo的git插件：

```powershell
npm install hexo-deployer-git --save
```

#### 修改Hexo中的`\_config.yml`文件：

```powershell
deploy:
  type: git
  repo: <repository url> # https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
 
# 例如，我自己的仓库配置如下：
deploy:
  type: git
  repo: git@github.com:JBR-Bunjie/JBR-Bunjie.github.io.git
  branch: master
```

配置完成之后，我们便可以使用命令：

```powershell
hexo deploy
```

来直接将我们编写的Hexo文档推送到所有远端托管地址上了

> 在使用git来推送之前，你需要先完成本地git工具对远程仓库的权限等配置

## 安装别的博客框架

当然，你需要先选定一个博客框架才行

比如如果想要安装<a href="#AuroraOfficialSite">aurora</a>，我会选择直接去查阅<a href="#AuroraOfficialSite">aurora</a>的官方文档来了解[aurora](#AuroraOfficialSite)的具体配置项

<span id="AuroraOfficialSite">[Home | Hexo Aurora (tridiamond.tech)](https://aurora.tridiamond.tech/)</span>

但是当我们想要更替当前Hexo框架时，我们需要同时修改Hexo项目的`\_config.yml`和Hexo框架的的`\_config.xxx.yml`，例如：Aurora的`\_config.aurora.yml`

```yaml
# 修改项：

theme: <yourThemeName>  
```

## 让Hexo支持Latex表达式

即便是aurora这种并非原生支持Latex的theme，也可以通过直接改变hexo的

```powershell
npm uninstall hexo-renderer-marked --save 

# 注意，pandoc并非渲染引擎本身，我们需要先下载并安装pandoc.exe，之后hexo-renderer-pandoc才能通过调用pandoc.exe来生成页面
npm install hexo-renderer-pandoc --save
npm install mathjex --save
```

当安装完成后，并且我们在对应页面的head中插入：

```markdown
mathjex: true
```

后，我们便可以通过正常的生成语句来得到正确显示的Latex公式了！

```powershell
# 以上所有步骤完成后。

hexo g
hexo s
```

<iframe src="//player.bilibili.com/player.html?aid=673965978&bvid=BV1xU4y1V7WU&cid=363620657&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
