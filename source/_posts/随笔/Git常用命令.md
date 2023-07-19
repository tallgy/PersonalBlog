---
title: Git常用命令
date: 2021-10-19 14:10:15
tags:
 - Git
 - 随笔
categories:
 - 随笔
---



#  Git常用命令



## 新建一个git仓库 git init

```
git init
```



## 将内容进行本地提交 git add

```
git add .
```



## 设置提交到GitHub的url git remote add

```
git remote add origin https://xxx
```



## 提交 git push

第一次需要这样使用，后续可以直接使用 git push

```
git push -u origin master
```



## 拉取 git pull

```
git pull origin master
```



## 同时将一个项目提交到两个仓库 git remote set-url

```
需要再添加一个新的仓库
git remote set-url --add origin https://xx
```

也可以在配置文件里面进行修改

.git/config

```
在文件里面的[remote "origin"]
下面添加一个url：设置为自己的另一个仓库的url地址就可以在提交的时候将仓库提交到远程了
```

## 添加 git 的 ssh 密钥

https://zhuanlan.zhihu.com/p/576717964



## git fork clone别人的仓库

https://blog.csdn.net/liuqiao0327/article/details/110671028
