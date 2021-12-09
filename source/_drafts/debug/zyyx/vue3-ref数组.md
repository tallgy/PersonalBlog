---
layout: draft
title: vue3-ref数组
date: 2021-12-09 13:35:22
tags:
 - bug
categories:
 - bug
---



#  Vue3 的 ref数组

## 问题

​		在vue2中，对于循环使用的ref，此时ref的值相同就会形成一个数组

```
<tags-select-group v-for="(item,index) in eventMaps" ref="tagsSelectGroup" />
```

​		但是对于Vue3来说，循环的ref并不能生成数组，如果需要使用ref数组，需要使用方法，将ref存入一个数组里面。

```
<tags-select-group v-for="(item,index) in eventMaps" :ref="setTagsSelectGroup"/>

data: {
	tagsSelectGroup: [],
},
beforeUpdate() {
	//在每次更新之前，需要将数组进行清零，避免重复进行push。
	this.tagsSelectGroup.length = 0
},
methods: {
	//这个默认不传递参数时使用的参数是 event，所以可以直接使用。当然，也有可能是vue3内部进行了额外的操作？
	setTagsSelectGroup(el) {
            el && this.tagsSelectGroup.push(el)
    },
}
```

