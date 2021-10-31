---
title: Node JS 读取文件
date: 2021/10/14
tags: 
 - Node JS
 - fs
categories:
 - Node JS
 - fs
---



# Node JS 读取文件



在  ` Node JS ` 中文件 操作是使用的 ` fs ` 模块进行了读取操作。



#### 使用` Node JS `读取文件

```
http://nodejs.cn/api/fs.html#fs_fs_readfile_path_options_callback
```

- `path` \<string> | \<Buffer> | \<URL> | \<integer> 文件名或文件描述符
- options   \<Object> | \<string>
    - `encoding` \<null> | \<string> **默认值:** `null`
    - `flag` \<string>  **默认值:** `'r'`。
    - `signal` \<AbortSignal>允许中止正在进行的读取文件
- callback   \<Function>
    - `err`  \<Error> | \<AggregateError>
    - `data` \<string> | \<Buffer>



```
const fs = require('fs');

fs.readFile('./../source/Node JS 读取文件.md', 'utf8', (err, data) => {
  if (err) {
    throw err;
  } else {
  	//data 是一个 十六进制的buffer，所以使用toString转成字符。
    console.log(data.toString());
  }
})
```



