---
title: webpack性能优化
date: 2023-07-11 17:41:07
tags:
 - webpack
categories:
 - webpack
---


# 减少 webpack 打包时间


## 优化 babel 的打包时间：

* 优化 babel 的文件范围，减少 babel 转移的文件量


* 缓存编译过的文件，下次编译只需要编译更改的文件即可
大部分 Loader 都提供了 cache 配置项，比如在 babel-loader 中，可以通过设置 cacheDirectory 来开启缓存，这样，babel-loader 就会将每次的编译结果写进硬盘文件（默认是在项目根目录下的node_modules/.cache/babel-loader目录内，当然你也可以自定义）


## happyPack

开启多线程打包


## DllPlugin	/	 Externals	抽离

DllPlugin 可以将特定的类库提前打包然后引入。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案。

## 代码压缩

在 Webpack3 中，我们一般使用 UglifyJS 来压缩代码，但是这个是单线程运行的，为了加快效率，我们可以使用 webpack-parallel-uglify-plugin 来并行运行 UglifyJS，从而提高效率。


# 减少 webpack 打包体积


## 按需加载

Scope Hoisting 会分析出模块之间的依赖关系，尽可能的把打包出来的模块合并到一个函数中去。如果在 Webpack4 中你希望开启这个功能，只需要启用 optimization.concatenateModules 就可以了。

Tree Shaking 可以实现删除项目中未被引用的代码。如果你使用 Webpack 4 的话，开启生产环境就会自动启动这个优化功能





