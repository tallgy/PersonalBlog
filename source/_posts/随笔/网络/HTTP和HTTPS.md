---
title: HTTP和HTTPS
date: 2021-11-05 21:52:44
tags:
 - HTTP
categories:
 - 随笔
---



#  HTTP和HTTPS的区别

参考链接

```
https://www.nowcoder.com/tutorial/96/4700c6f1f3334c9191a38406002efa65
```



## 体系结构

首先我们要知道 OSI网络参考模型 和 TCP/IP模型 以及最后的五层协议的体系

<img src="http和https/image-20210831164301870.png" alt="image-20210831164301870" style="zoom:67%;" />

**对比**

* TCP/IP 将OSI 应用层，表示层，会话层 合并为了 应用层，但是同时也将 数据链路层 和物理层合并为了网络接口层。
* 而五层协议在TCP/IP和OSI的综合下，合并了应用层，但是也保留了数据链路层和物理层



在这里，应用层就是HTTP的部分了。



## HTTP协议

​		HTTP：超文本传输协议。是一个客户端和服务器端应答的标准(TCP)，



### HTTP报文格式

​		请求行，请求头，空行，请求体



## HTTPS

​		HTTPS，在HTTP的基础上，添加了安全，因为HTTP都是进行的明文传输。

​		HTTPS的SSL加密是在传输层实现的。

​		HTTPS的主要作用是：建立一个信息安全通道，来确保数组的传输，确保网站的真实性。



### SSL/TLS

SSL，安全套接层，Secure Sockets Layer

TLS，安全传输层，Transport Layer Security



### 摘要算法

* 它可以把任意长度的数据压缩成固定长度，并且独一无二的摘要字符串。
* 通过把明文信息和摘要一起加密传输，接收后解密再对明文信息进行摘要，判断是否被修改。



### 对称加密和非对称加密

* 对称加密，加密和解密使用相同的密钥进行实现
* 非对称加密，有两个密钥，一个是**私钥**，一个是**公钥**，**私钥是必须严格保密的，用于解密的。公钥是公开的，进行加密的。** 



### 混合加密

* 因为非对称加密的传输速度较慢，所以使用混合加密的方式。

* 简而言之，通过非对称加密进行传输对称加密的密钥，然后通过对称加密进行数据传输。



### 数字证书

* 对于一个服务是否可信，我们通过使用了数字证书来进行验证。
* 里面包括了 CA信息，公钥用户的信息，公钥，权威机构的签名，有效期
* 数字证书的作用： 1.向浏览器证明身份。 2.里面含有公钥。



### HTTPS的传输步骤

* 客户端使用HTTPS进行访问时，则要求web服务器建立SSL连接。

* web服务端收到请求后，会将网站的数字证书返回给客户端。
* 客户端验证过后，开始和web服务器协商SSL连接的安全等级，就是加密等级。
* 客户端通过双方协商后的安全等级，建立会话密钥，然后通过网站的公钥来加密会话密钥，并传送给网站。
* 服务端进行解密，也获取了密钥
* 使用密钥进行传输



### HTTPS的优点

* 安全，防止数据在过程中不被窃取，改变，确保了数据的完整性。



### HTTPS的缺点

* 费时
* 缓存没有HTTP高效



## HTTP和HTTPS的区别

* HTTPS协议需要ca证书，费用较高。
* HTTP是超文本传输协议，信息是明文传输，HTTPS则是具有安全性的ssl加密传输协议。
* 使用不同的链接方式，端口也不同，一般而言，HTTP协议的端口为80，HTTPS的端口为443
* HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比HTTP协议安全。





