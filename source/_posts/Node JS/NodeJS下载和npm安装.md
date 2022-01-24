---
layout: draft
title: NodeJS下载和npm安装和配置
date: 2021-12-28 11:36:56
tags:
 - 安装教程
 - NodeJS
 - npm
 - 随笔
---



#  NodeJS下载和npm安装



下载 NodeJS，没啥，直接官网进入，下载包

```
https://nodejs.org/zh-cn/download/
```



一路安装，node 安装会自动安装 npm

安装完成之后

通过在 cmd 使用 来判断是否安装完成

```
npm -v
node -v
```



## 修改安装路径

### 查看路径

```
npm prefix -g
```



### 修改路径

```
这个是全局安装时的路径，使用全局安装会将其安装在这里，当然你也可以直接安装在nodejs这个目录就行，不需要更深层的目录。但是这样不方便查看和删除。
npm config set prefix “D:\Software\nodejs\node_global”
这个是缓存的路径，npm会生成缓存。用于更快的进行启动和操作。
npm config set cache “D:\Software\nodejs\node_cache”
```



配置环境变量

![image-20211228140948410](NodeJS下载和npm安装\image-20211228140948410-16406717947501.png)



然后点击确定保存即可。



### 配置镜像

我们先配置一个镜像然后再安装一个源管理工具nrm

这个是一个淘宝的镜像源

```
npm config set registry=https://registry.npm.taobao.org
```



### 安装nrm

如果出现操作错误，试试使用管理员的cmd进行操作

```
npm i -g nrm
```



### 使用nrm

```
nrm ls
输出nrm的一个镜像仓库

nrm use xxx
使用这个xxx镜像

nrm add name url
添加一个镜像仓库，名字是name，地址是url

nrm del xx
删除xx仓库
```



