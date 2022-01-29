---
title: sass预处理
date: 2021-10-27 16:05:34
tags:
 - css
 - sass
categories:
 - css
 - sass
---



#   安装sass

使用sass，需要安装ruby，然后再安装sass

如果是webpack这种项目的，可以使用 npm 下载包

```
"sass": "^1.49.0",
"sass-loader": "^7.1.0",
上面这两个下载了就可以编译了

还有一个是 node-sass，但是这个下载好像需要python环境，而且貌似不需要也可以使用？但是对于这个sass的安装和环境的一些我还是不算很是理解清楚。
```



## 第一步，安装 ruby

首先，我们得知道，Sass是基于Ruby开发的，所以要运行Sass都需要一个Ruby环境.

所以下面这个链接就是 ruby 的下载链接，如果下载速度慢，可以使用 迅雷来增加下载速度。

```
http://www.ruby-lang.org/en/downloads/
```

阿里云盘的链接， 阿里云盘应该挺快.

```
「rubyinstaller-devkit-2.7.5-1-x64.exe」https://www.aliyundrive.com/s/RXSr5Fh38uz
点击链接保存，或者复制本段内容，打开「阿里云盘」APP ，无需下载极速在线查看，视频原画倍速播放。
```



然后就是安装了

这里也直接给别人的安装链接

```
https://www.cnblogs.com/padding1015/p/7133811.html
```

**我就提几个重点**

* 要添加到 Path，这样可以直接在cmd里面进行运行。

* 在最后一步之前 finish  **ridk install**  这个去掉，因为没有使用镜像，会比较慢，要引入镜像

  * 参考链接

  * ```
    https://blog.csdn.net/mscf/article/details/82627951
    ```

  * 然后也就是 添加镜像，设置 gem 源了

* 我这里记录链接里面的重点，怕到时候链接没了

* ```
  双击开始安装，选择全部安装；
  在安装结束时，去除ridk install的选项，因为从默认的原去下载几百兆会非常缓慢；
  查找Ruby安装目录下的msys64\etc\pacman.d，编辑更新源：
  
  Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686 加入mirrorlist.mingw32首位
  Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64 加入mirrorlist.mingw64首位
  Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch 加入mirrorlist.msys首位
  
  执行在命令行执行ridk install（如果安装时选择了不加入系统环境变量的，去Ruby安装目录的bin之下执行），一路回车至结束；
  更新gem源：
  gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
  ```

  



## 然后就是安装sass

先 **gem env** 了解一下安装了路径

```
参考资料：
https://qastack.cn/programming/19072070/how-to-find-where-gem-files-are-installed

https://guides.rubygems.org/command-reference/#gem_environment
```

```
注意： INSTALLATION DIRECTORY
			GEM PATHS
```



一波团灭

```
gem install sass

然后准备回车就行了
```

可能要等一会儿，我开始等了一会儿，没有反应，不知道是不是连接镜像查找的时候太久了。

不出意外是可以成功的，因为国内镜像我们在上一个blog上面设置好了的





## 使用

这个，一般搜一下怎么配置就行了

我的是 webstorm，所以好像当时自动配好了



# 使用 sass

## 嵌套选择

```
.aaa {
  xxx
  
  .bbb {
  	zzz
  }
}
```

上面代码就是嵌套选择，就是代表下面的意思

```
.aaa {
	xxx
}
.aaa .bbb {
	zzz
}
```



## 定义变量

可以定义变量，然后进行使用，所以对于相同属性的，可以直接使用变量，后续的修改也只需要修改一个就行。

```
$padding-sm: 5px;

.a {
	padding: $padding-sm;
}
```





