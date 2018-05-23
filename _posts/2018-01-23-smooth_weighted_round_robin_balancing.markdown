---
layout: post
title: "平滑的加权轮询算法"
subtitle: ""
date: 2018-01-23
author: LuJiangBo
tags: 算法
keyword: 算法,数据结构
finished: true
---
# 轮询算法

在讨论如何实现[负载均衡](https://zh.wikipedia.org/wiki/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)时,我们很容易就能甩出一大堆常见的算法，比如轮询法，随机法，源地址哈希法，最小连接数法等等。而这里我要总结的是轮询法的进阶版**平滑的加权轮询算法**。

## 什么叫轮询
 
基于维基百科的介绍，[轮询](https://zh.wikipedia.org/wiki/%E8%BC%AA%E8%A9%A2)(Polling)是一种CPU决策如何提供周边设备服务的方式，又称“程控输入输出”（Programmed I/O）。轮询法的概念是：由CPU定时发出询问，依序询问每一个周边设备是否需要其服务，有即给予服务，服务结束后再问下一个周边，接着不断周而复始。也就是说在我们的场景中就是给定一组服务器列表，依次的将每一个进来的请求轮流的分配给列表中的每一台服务器来实现负载均衡。
* 优点：实现容易,每台服务器请求数相同(￣□￣｜｜好像也就这一个优点了)
* 确定：效率偏低,无法满足服务器配置不同的情况(说好的能者多劳呢)

## 什么叫普通的加权轮询

基于上面简单轮询所带来的问题，就有了轮询的变种**加权轮询**。  
所谓的加权轮询也就是在配置服务器列表时，给每一台服务器配置一个权重值。
举个例子：现在有3台服务器(A:3)(B:2)(C:1)，数字分别代表它们的权重值，数字越大表示所能承受的压力越大。将3台服务器的权重值相加3+2+1=6,也就是说现在每6个请求进来其中会有3个分配个A，2个分配个B，剩下的一个分配给C，依次循环A-A-A-B-B-C-A-A-A-B-B-C-A.  
* 优点：根据权重，可将性能更优越的服务器分配更多的请求数
* 缺点：因为是按顺序依次轮询将请求分配给服务器，所以权重大的服务器会在单位时间内分配到权重比例的请求数，这并不是一种均匀的分配方法。

## 平滑的加权轮询算法

鉴于普通的加权轮询算法存在的问题，有了更好的平滑的加权轮询算法，在该算法中，对于权重情况{3,1,2},会得到这样一组结果{ACABCA}而不再是{AAABCC}.
这方面运用的比较成功的一个开源项目即Nginx中的[加权轮询算法](https://github.com/phusion/nginx/commit/27e94984486058d73157038f7950a0a36ecc6e35)，


## 算法说明

算法步骤：  
* 首先每个节点分别有**CurrentWeight**(当前权重值)，**EffectiveWeight**(有效权重值)，**Weight**(配置权重值)三个权重字段，
其中Weight即用户配置的权重值，EffectiveWeight初始为取Weight值，在实际运行中会根据服务器的失败情况所相应的增减，CurrentWeight即当前服务器权重值，初始取EffectiveWeight值
* 在选择服务器节点时，每次取CurrentWeight最大的一项，获取到之后再将该节点的CurrentWeight值改为(CurrentWeight=CurrentWeight-total)即，当前权重等于当前权重减去总权重，而总权重又是所有节点的有效权重值相加。
下图是取自Github Nginx项目提交说明：
![平滑的加权轮询算法]({{ post.url| prepend: site.url  }}/content/images/201801/smooth_weighted_round_robin_balancing01.png)

## 算法实现

需要说明的是，我这里的代码是基于Nginx开源实现而改写的一个golang版本的简要版，省略了失败降权等优化操作，写出平滑加权轮询的基本操作。有兴趣的同学也可以直接去看Nginx的源码，大概位置是[ngx_http_upstream_round_robin.c](https://github.com/phusion/nginx/blob/master/src/http/ngx_http_upstream_round_robin.c)和[ngx_http_upstream_round_robin.h](https://github.com/phusion/nginx/blob/master/src/http/ngx_http_upstream_round_robin.h)这2个文件

```
package main

import (
	"log"
)

//Peer 单个配置节点
type Peer struct {
	Sockaddr        string //id地址
	CurrentWeight   int    //当前权重
	EffectiveWeight int    //有效权重
	Weight          int    //配置权重
}

//Peers 服务器地址池
type Peers struct {
	number      int     //服务器数量
	TotalWeight int     //总权重
	peer        []*Peer //服务器节点数组
}

//PeerData 解析后的服务器数据
type PeerData struct {
	Peers *Peers //服务器池数据
}

//ServerConfig 模拟配置文件
//配置节点
type ServerConfig struct {
	Addr   string
	Weight int
}

//InitPeer 节点初始化
func InitPeer(cf []ServerConfig) *PeerData {
	pd := new(PeerData)
	pd.Peers = new(Peers)
	c := len(cf)
	pd.Peers.number = c
	pd.Peers.peer = make([]*Peer, c, c)
	for i := 0; i < pd.Peers.number; i++ {
		pd.Peers.peer[i] = new(Peer)
		pd.Peers.peer[i].Weight = cf[i].Weight
		pd.Peers.peer[i].Sockaddr = cf[i].Addr
		pd.Peers.peer[i].EffectiveWeight = cf[i].Weight //将EffectiveWeight初始为Weight值
	}
	return pd
}

//GetPeer 获取一台节点地址
func GetPeer(rrp *PeerData) *Peer {
	var best *Peer
	total := 0

	for i := 0; i < rrp.Peers.number; i++ {
		peer := rrp.Peers.peer[i]
		peer.CurrentWeight += peer.EffectiveWeight //将当前权重与有效权重相加

		total += peer.EffectiveWeight //累加总权重

		if best == nil || peer.CurrentWeight > best.CurrentWeight {
			best = peer
		}

	}
	if best == nil {
		return nil
	}
	best.CurrentWeight -= total //将当前权重改为当前权重-总权重
	return best
}

func main() {
	//测试数据，模拟服务器配置列表
	testData := []ServerConfig{
		ServerConfig{
			Addr:   "a",
			Weight: 3,
		}, ServerConfig{
			Addr:   "b",
			Weight: 1,
		}, ServerConfig{
			Addr:   "c",
			Weight: 2,
		},
	}
	pd := InitPeer(testData)
	//模拟12次请求进入
	for i := 0; i < 12; i++ {
		p := GetPeer(pd)
		log.Println(p.Sockaddr)
	}
}



```