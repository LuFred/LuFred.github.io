---
layout: post
title: ".Net Core 引用自定义类库"
subtitle: ".Net Core中引用自定义类库"
subtitleAuthor: ""
date: 2016-12-25
author: LuJiangBo
tags: DotNet Core
keyword: dotnet core
finished: true
---


## 问题

&emsp;&emsp;.Net Core多项目开发类库引用问题。

## 准备
&emsp;&emsp;假设现在有如下2个项目。一个Host,一个Core，其中Host依赖Core。  
	![项目]({{ post.url| prepend: site.url  }}/content/images/201612/2016-12-25-dotnetaddreference.png) 
    
## 引用Crawler.Core
![host]({{ post.url| prepend: site.url  }}/content/images/201612/2016-12-25-dotnetaddreference01.png)  
打开Host项目下的**project.json**文件，在**dependencies**节点下添加Crawler.Core,其中需要注意的是 "target": "project" 这个属性，它表示引用的是项目而不是nuget包，这和前面的Microsoft.NETCore.App 的依赖属性"type": "platform"类似。  

##### 记得最后项目要重新restore一下。



