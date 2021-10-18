---
title: hexo基础
date: 2021/10/14
tags: hexo
---



# hexo

md文件和html的文件对应关系在那个 ` db.json `， 它使用了hash，id以及其他，我这里就使用简单的形式对应就行了。



# 使用主题

我这里使用的是next主题。这里有两个，一个应该是以前个人制作的。第二个是有一个组织进行了维护吧，具体没有了解过。

```
http://theme-next.iissnan.com/
```

```
https://theme-next.js.org/
```

这里可能会因为新的版本的原因网页不能显示，这里我去百度了。

原因是hexo在5.0之后把swig给删除了需要自己手动安装，所以只需要安装一下 这个就行了。

```java
 npm i hexo-renderer-swig
```



# 基本操作

## 写作

```
https://hexo.io/zh-cn/docs/writing
```

```
layout， 
post， 会发表的
draft， 草稿，不会发表的。私密的
page， 就是页面，会在source文件里面生成你写的文件名字，然后在里面生成一个index.md，然后，你可以在你的页面上输入 url/文件夹的名字就可以访问了，用于写页面，首页，关于，标签等都是使用的这个方法。
```



## 标签插件（Tag Plugins）

直接在md文档上写就完了。会自动转译的。

