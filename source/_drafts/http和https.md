---
title: http和https
date: 2021-11-05 21:52:44
tags:
 - HTTP
 - HTTPS
categories:
 - HTTP
---



#  HTTP和HTTPS的区别

## 体系结构

首先我们要知道 OSI网络参考模型 和 TCP/IP模型 以及最后的五层协议的体系

<img src="http和https/image-20210831164301870.png" alt="image-20210831164301870" style="zoom:67%;" />

**对比**

* TCP/IP 将OSI 应用层，表示层，会话层 合并为了 应用层，但是同时也将 数据链路层 和物理层合并为了网络接口层。
* 而五层协议在TCP/IP和OSI的综合下，合并了应用层，但是也保留了数据链路层和物理层



在这里，应用层就是HTTP的部分了。



## HTTP协议

### HTTP报文格式

请求行，请求头，空行，请求体

HTTP使用的TCP三次握手。



## HTTPS

因为HTTP是明文传输，所以会有很多问题的产生，所以此时就诞生了HTTPS，HTTPS的区别在于应用层和传输层之间加了一个属于应用层的模块，HTTP和TCP连接之间加了SSL/TSL

<img src="http和https/image-20210809222623510.png" alt="image-20210809222623510" style="zoom:67%;" />



