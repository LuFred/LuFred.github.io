---
layout: post
title: "gRPC中的负载均衡"
subtitle: ""
date: 2018-05-23
author: LuJiangBo
tags: gRPC 
keyword: gRPC
finished: true
---

# 背景

## 基于请求的负载均衡

值得注意的是，gRPC内实现的负载均衡是基于每个请求而不是每个连接。 换句话说，即使所有请求都来自单个客户端，其仍然希望它们在所有服务器之间进行负载均衡。

## 负载均衡的方法

在讨论任何gRPC细节之前，首先需要探讨一些常用的实现负载均衡的方法。

### 代理模型

使用代理提供一个可靠的客户端，可以向负载均衡系统报告负载。代理通常需要更多的资源来操作，因为它们具有RPC请求和响应的临时副本。 该模型也增加了RPC的延迟。  

在考虑像存储这样的重要服务时，代理模型被认为是低效的。

### 客户端负载均衡

该方法是在客户端中放置了更多的负载均衡逻辑。例如，客户端可以包含许多用于从列表中选择服务器的负载均衡策略（循环，随机等）。在此模型中，服务器列表将在客户端进行静态配置，由名称解析系统，外部负载均衡器等提供。在任何情况下，客户端都负责从列表中选择首选服务器。

这种方法的缺点之一是以多种语言/客户端版本编写和维护负载均衡策略。这些政策可能相当复杂。某些算法还需要客户端到服务器的通信，因此除了为用户请求发送RPC外，客户端需要变得更复杂以支持额外的RPC以获取运行状况或加载信息。

这也会使客户端代码复杂化：新的设计隐藏了多个层的负载均衡复杂性，并将其作为一个简单的服务器列表呈现给客户端。

### 外部负载均衡服务

客户端负载均衡代码保持简单和可移植，实现常见的用于服务器选择的算法(如轮询调度)。负载均衡器提供了复杂的负载均衡算法。客户端依赖负载均衡器来提供负载均衡配置和客户端应该发送请求的服务器列表。均衡器更新服务器列表，以均衡负载和处理服务器的不可用性或健康问题。负载均衡器将做出任何必要的复杂决策并通知客户。 负载均衡器可以与后端服务器通信以收集负载和健康信息

# 要求

## 简单的客户端和API

gRPC客户端负载均衡代码必须简单且可移植。 客户端应该只包含用于服务器选择的简单算法（例如循环）。 对于复杂的算法，客户端应该依靠负载均衡器来提供负载均衡配置以及客户端应该向其发送请求的服务器列表。 均衡器将根据需要更新服务器列表，以均衡负载以及处理服务器不可用性或健康问题。 负载均衡器将做出任何必要的复杂决策并通知客户。 负载均衡器可以与后端服务器通信以收集负载和健康信息。

## 安全

负载均衡器可能与实际的服务器后端分离，而负载均衡器的妥协只会导致负载均衡功能的损害。 换句话说，受损的负载均衡器不应导致客户端信任（可能是恶意的）后端服务器，而不是在没有负载均衡的情况下进行比较。

# 架构

## 概要

gRPC中负载均衡的主要机制是外部负载均衡，其中外部负载均衡器为简单的客户端提供最新的服务器列表。

gRPC客户端确实支持内置负载平衡策略的API。但是，其中只有少数（其中一个是grpclb实施外部负载平衡的 策略），并且不鼓励用户尝试通过添加更多来扩展gRPC。相反，应在外部负载平衡器中实施新的负载平衡策略。

## 工作流程

在名称解析和连接到服务器之间，负载平衡策略适合gRPC客户端工作流程。以下是它的全部工作原理：
![工作流程]({{ post.url| prepend: site.url  }}/content/images/201805/grpc_load_balancing.png)

* 1.在启动时，gRPC客户端为服务器名称发出[名称解析](https://github.com/grpc/grpc/blob/master/doc/naming.md)请求。名称将解析为一个或多个IP地址，每一个都将指示它是一个服务器地址还是一个负载平衡器地址，以及一个服务配置，它指示使用哪个客户端负载平衡策略(例如，round_robin或grpclb)。  
* 2.客户端实例化负载平衡策略。  
    * 注意:如果解析器返回的任何一个地址是平衡器地址，则客户端将使用grpclb策略，而不管服务配置请求什么负载平衡策略。 否则，客户端将使用服务配置请求的负载平衡策略。 如果服务配置没有请求负载平衡策略，那么客户端将默认选择一个策略来选择第一个可用的服务器地址。
* 3.负载平衡策略为每个服务器地址创建一个子通道。  
    * 对于grpclb以外的所有策略，这意味着解析器返回的每个地址都有一个子通道。 请注意，这些策略会忽略解析器返回的任何平衡器地址。
    * 在grpclb策略的情况下，工作流如下:  
        * 该策略将打开流解析器返回的其中一个平衡器地址的流。 它向平衡器请求服务器地址用于客户端最初请求的服务器名称（即，最初传递给名称解析器的同一个服务器名称）。
            * 注意：在grpclb策略中，解析器返回的非平衡器地址用作回退，以防LB策略启动时无法联系到平衡器。
        * 如果负载平衡器配置需要该信息，负载平衡器将客户端指向的gRPC服务器可向负载平衡器报告负载。
        * 负载均衡器将服务器列表返回给gRPC客户端的grpclb策略。 然后，grpclb策略将为列表中的每个服务器创建一个子通道。
    * 对于每个发送的RPC，负载均衡策略决定应该将RPC发送到哪个子通道（即哪个服务器）。
        * 对于grpclb策略，客户端将按照负载均衡器返回的顺序向服务器发送请求。 如果服务器列表为空，则呼叫将阻塞，直到收到非空的呼叫。


[原文](https://github.com/grpc/grpc/blob/master/doc/load-balancing.md)