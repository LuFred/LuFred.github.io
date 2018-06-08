---
layout: post
title: "TCP套接字编程基本函数介绍"
subtitle: ""
date: 2018-05-23
author: LuJiangBo
tags: tcp unix socket 
keyword: tcp,unix,socket
finished: false
---

总结TCP客户端/服务器编程所需要的基本的套接字函数

## socket函数

要执行网络I/O，一个进程必须要调用socket函数，来指定期望的通信协议类型
```
#include <sys/socket.h>
int socket(int family,int type,int protocol);
                            返回:若成功则为非负描述符,若出错则返回-1
```
参数介绍:
* family:协议族  

| family      | 说明|
| --------    | -----|
| AF_INET     | IPv4协议|
| AF_INET6    | IPv6协议|
| AF_LOCAL    | Unix域协议|
| AF_ROUTE    | 路由套接字|
| AF_KEY      | 秘钥套接字|

* type:套接字类型  

| type      | 说明|
| --------    | -----|
| SOCK_STREAM       | 字节流套接字|
| SOCK_DGRAM        | 数据报套接字|
| SOCK_SEQPACKET    | 有序分组套接字|
| SOCK_RAW          | 原始套接字|

* protocol:某个协议类型常值，或者设为0，以选择所给定的*family*和*type*组合的系统默认值

## connect函数

TCP客户端利用connect函数来建立与TCP服务器的连接。
```
#include <sys/socket.h>
int connect(int sockfd,const struct sockaddr *servaddr,socklen_t addrlen);
                            返回：成功:1,失败:-1
```
参数介绍:
* sockfd:socket函数返回的套接字描述符
* *servaddr:套接字地址结构的指针
* addrlen:套接字地址结构的大小   
如果是TCP调用,几种可能的返回错误情况:
* 1.若TCP客户没有收到SYN分节响应，则返回ETIMEDOUT错误。且会按照(书TCPv2)一点规则,6s、24s、75s重试，若仍未收到则返回错误。
* 2.若客户端的SYN的响应是RST(表示复位),则表示服务器主机在我们制定的端口上没有进程在等待与之连接，这是一种硬错误(hard error)，客户端一接收到RST直接返回ECONNREFUSED错误。
* 3.若客户端发出的SYN在中间某个路由器上引发一个"destination unreachable"目的不可达的ICMP错误，则认为是一种软错误(soft error)，则客户主机内核保存该消息，并按照第一种情况中所述的时间间隔继续发送SYN,若在某个规定时间后仍未收到响应，则把保存的消息作为EHOSTUNREACH或ENETUNREACH错误返回给进程。  

```
ps:若connect失败则该套接字不再可用，必须关闭，我们不能对这样的套接字再次调用connect函数。

ps:客户端再调用函数connect前不必非得调用bind函数。因为如果需要,内核会确定源IP地址，并选择一个临时端口作为源口。 
```

## bind函数

