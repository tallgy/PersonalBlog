---
title: hexo基础
date: 2021-10-13 16:12:42
tags: 
 - 随笔
categories:
 - 随笔
---



# hexo

md文件和html的文件对应关系在那个 ` db.json `， 它使用了hash，id以及其他。

```
https://hexo.io/zh-cn/docs/
```



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



## 图片上传

```
https://blog.csdn.net/xjm850552586/article/details/84101345
```

使用插件  `hexo-asset-image`



## 上传

```
npx hexo clean
	清除缓存文件 ( db.json) 和生成的文件 ( public)。
npx hexo g
	生成静态文件
nex hexo d
	将静态文件上传
```

```
这里因为需要使用三个命令，很麻烦，所以就想着简单化一点。
```

```
开始是想着使用npm或者npx命令，自己写一个的，但是因为技术不够，所以还不会，后面了解了更多了之后再进行解决。
```

```
所以，我是用的方式是，通过一个bat文件直接将这三个命令写下来，我只需要运行这个文件就行了。
所以我在blog里面创建了一个 
publish.bat
然后直接在里面，写入

call npx hexo clean
call npx hexo g
call npx hexo d

这里需要在前面加上call方法。在批处理脚本中，call命令用来从一个批处理脚本中调用另一个批处理脚本。
就这样就完成了。。
```



## gitee自动更新

```
使用puppeteer模块

但是因为我谷歌浏览器修改过位置，所以需要重新下载，所以我没有进行后续的操作了。

https://www.jianshu.com/p/6460df84a099
```

