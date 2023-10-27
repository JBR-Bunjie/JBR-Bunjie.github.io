---
title: NPM 基本命令
date: 2021-1-31 5:23:23
tags:
  - Javascripts
categories:
  - Javascripts
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# NPM 基本命令

## 修改 npm 源

### 设置 npm 下载来源为淘宝源

1. 查看当前源

```powershell
npm config get registry
```

2. 设置为淘宝源

```powershell
npm config set registry https://registry.npm.taobao.org
```

3. 临时使用源（适用于一次性使用）

```powershell
npm --registry https://registry.npm.taobao.org install XXX（module name）
```

### 还原默认源：

```powershell
npm config set registry https://registry.npmjs.org/
```

### 采用 cnpm

- cnpm 介绍：
  淘宝搭建的一个国内的 npm 服务器，以 10 分钟每次的速度将 npm 仓库中的所有内容“搬运”到国内
- cnpm 安装：
  npm install -g cnpm --registry=https://registry.npm.taobao.org
- cnpm 与 npm 的关系：cnpm 就是 npm 的另一个版本，是 node 中一个不同的下载器，只包含了 npm 的下载功能

### nrm

- nrm 介绍：
  nrm 是 npm 注册表的管理工具，可以添加、删除、查询、切换 npm 注册表。 什么是 nrm nrm 是一个 npm 源管理器，允许你快速地在 npm 源间切换。
- nrm 简单使用：

## 修改 NPM 的缓存以及全局安装地址：

1. 查看设置

```powershell
npm config ls
```

2. 设置全局安装地址：

```powershell
npm config set prefix "D:/npm/npm_Download"
```

3. 设置缓存地址

```powershell
npm config set cache "D:/npm/npm_Cache"
```

4. 清除缓存：

````powershell
npm cache clean --force
	```npm WARN using --force I sure hope you know what you are doing.
````

## NPM 管理安装包：

1. 查看已经安装的包：

```powershell
npm list -g --depth 0
```

2. 更新包：

```powershell
npm update -g xxx
```

3. 卸载包：

```powershell
npm uninstall -g xxx
```
