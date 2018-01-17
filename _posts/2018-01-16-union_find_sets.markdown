---
layout: post
title: "并查集"
subtitle: ""
date: 2018-01-16
author: LuJiangBo
tags: 算法
keyword: 排序,算法,数据结构
finished: true
---
# 并查集

并查集是一种树形结构，它的作用是用来出来一些不相交集的合并和查询问题。因此在讨论它之前，我们需要知道什么叫做不相交集。

## 不相交集

对于任何一对元素(a,b),a,b∈S，aRb为false或true。
在数学中有这样一种等价关系，若满足下列的三个性质的关系R：
* (自反性)对于所有的a∈S,aRa。
* (对称性)若aRb当且仅当bRa。
* (传递性)若aRb 且 bRc 则 aRc。
我们称a,b是等价的，即相交的，反之则是不相交的。

## 并查集

明确了什么叫做不相交集，我们就可以定义出并查集再这里的作用：  
* Find：确定元素属于哪一个子集。它可以被用来确定两个元素是否属于同一子集
* Union：将两个子集合并成同一个集合

并查集是一种树形结构，所以这里的Find查找的是节点所对应是树根节点，而Union的合并即将其中一棵树的根节点合并到另一颗数的根节点中去，作为它的子节点。  
下图取自维基百科的动图，它展示了并查集的作用：
![并查集]({{ post.url| prepend: site.url  }}/content/images/201801/union_find_sets01.png)

## Find查找

最简单的方法我们只需要使用递归方法从被找节点一层一层的寻找到它的父节点直到找到根节点为止，因为是树形结构，所以所花费的时间并不会太大O(logN)
```

//Find 简单查找
func Find(arr []int, x int) int {
	if arr[x] == x {
		return x
	}
	return Find(arr, arr[x])
}

```
但是我们在这需要考虑的一个问题是，寻找父节点时采用递归，但是一旦元素一多起来，或退化成一条链，每次Find都将会使用O(n)的复杂度，这显然不是我们想要的。因此我们需要在查询的同时进行路径压缩，即当找到根节点是，顺便把它的子孙(递归时所经过的子节点)直接连接到它上面。使用路径压缩的代码如下：
```

//FindC 路径压缩查找
//操作：使从X到根的路径上的每一个节点都使它的父节点变成根节点
func FindC(arr []int, x int) int {
	if arr[x] == x {
		return x
	}
	arr[x] = Find(arr, arr[x])
	return arr[x]
}

```

## Union合并
合并一般有3种实现方式  
  
1.简单合并，即将后一个元素合并到前一个元素中  
它容易出现一种坏结果，即最后的树是一条链

```
//Unit 
func Unit(arr []int, r1, r2 int) {
	if r1 == r2 {
		return
	}
	arr[r2] = r1
}

```

2.按高度(有时也称为秩——rank)合并，即树高小的往大的合  
它能保证任何节点的深度均不会超过logN
```

//Unit
func Unit(arr []int, r1, r2 int) {
	if arr[r1] < arr[r2] {
		arr[r2] = r1
	} else {
		if arr[r1] == arr[r2] {
			arr[r2]--
		}
		arr[r1] = r2
	}
}

```

3.按大小合并,即元素总数小的往大的合  
它同样能保证任何节点的深度均不会超过logN
```

//Unit
func Unit(arr []int, r1, r2 int) {
	if r1 == r2 {
		return
	}
	x, y := arr[r1], arr[r2]
	if x < y {
		arr[r2] = r1
		arr[r1] = x + y
	} else {
		arr[r1] = r2
		arr[r2] = x + y
	}

}

```

## 时间复杂度

同时使用路径压缩、按秩合（rank）并优化的程序每个操作的最坏时间几乎是线性的，平均时间仅为O(α(M,N))，其中α(M,N)是n=f(x)=A(M,N)的反函数，而A是急速增加的[阿克曼函数](https://zh.wikipedia.org/wiki/%E9%98%BF%E5%85%8B%E6%9B%BC%E5%87%BD%E6%95%B8)。因为α(M,N)是其反函数，所以其在n十分巨大时还是小于5(因为A(4,1)的数量级已经约等于2的65536次方一个20000位的大数).因此平均运行时间是一个极小的常数。