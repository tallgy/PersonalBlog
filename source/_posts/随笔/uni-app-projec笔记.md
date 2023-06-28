---
title: uni-app-projec笔记
date: 2022-01-08 16:44:02
tags:
 - 随笔
 - 项目
 - 微信小程序
 - uniapp
categories:
 - 随笔
---



# uni-app 项目开发笔记问题

## 不要挂载原型链

将对象挂载在原型链是一个大忌，并且vue2也有警告，因为这样对于维护是一个非常大的问题，使用 forin会将内容显示出来

```
Object.prototype.a = {}
```



## sass的使用

对于根项目的 uni.scss 我们在使用的时候不需要进行引入。

项目的 node_modules vue-cli-plugin-uni/lib/options.js 里面有引入

```
https://uniapp.dcloud.io/collocation/uni-scss
```



## tabBar

微信小程序的tabBar可以使用 page.json里面进行配置，不需要自己定义组件

```
https://uniapp.dcloud.io/collocation/pages?id=tabbar
```

```
{
	"tabBar": {
		"color": "#808080",
		"selectedColor": "#007aff",
		"backgroundColor": "#f8f8f8",
		"list": [
			{
				"text": "主页",
				"iconPath": "static/img/tab-bar/tab-bar-unselect.png",
				"selectedIconPath": "static/img/tab-bar/tab-bar-select.png"
			}
		]
	},
}
```

几个比较重要的属性

```
文字颜色
color
selectedColor
背景颜色
backgroundColor

tab列表，是一个数组
list
```

```
list
是一个数组，每个元素都是对象

pagePath
text
iconPath
selectedIconPath
```



## 路由 router

微信小程序 没有路由

对应的在 pages.json 的 pages 里面可以进行配置，

```
"pages": [ //pages数组中第一项表示应用启动页，参考：https://uniapp.dcloud.io/collocation/pages
  {
    "path": "pages/index/index",
    "style": {
      "navigationBarTitleText": "uni-app"
    }
  }
],
```

