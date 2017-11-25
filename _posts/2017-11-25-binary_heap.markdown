---
layout: post
title: "二叉堆"
subtitle: "优先队列-二叉堆"
date: 2017-11-25
author: LuJiangBo
tags: 算法
keyword: 算法,数据结构
finished: true
---

## 简介
二叉堆是一种特殊的[堆](https://zh.wikipedia.org/zh-hans/%E5%A0%86_(%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84))，二叉堆是完全[二叉树](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%A0%91)或者是近似完全二叉树。  
二叉堆满足堆特性：
> 父节点的键值总是保持固定的序关系于任何一个子节点的键值，且每个节点的左子树和右子树都是一个二叉堆。

## 结构性质
堆是一颗被完全填满的二叉树，有可能堆例外是底层，底层上堆元素从左到右填入。  
下图就是一棵完全二叉树。  
![完全二叉树]({{ post.url| prepend: site.url  }}/content/images/201711/binary_heap01.png) 
我们可以用一个数组来表示完全二叉树，对于数组中的任一位置i上的元素，其左儿子在位置2i上，右儿子在左儿子后的单元(2i+1)中，它的父亲则在位置[i/2]上。

## 堆序性质
使操作被快速执行的性质是堆序性。在一个堆中，对于每一个节点X，X的父亲中的关键字小于(或等于)X中的关键字，根节点除外(它没有父亲)。  
二叉堆有两种：最大堆和最小堆。
 * 最大堆：父结点的键值总是大于或等于任何一个子节点的键值；
 * 最小堆：父结点的键值总是小于或等于任何一个子节点的键值。


## 基本操作
基本数据结构定义
```
//HeapStruct 堆结构
type HeapStruct struct {
	Size     int32   //当前元素个数
	Capacity int32   //堆容量
	Elements []int32 //堆数组
}
var (
	MaxHeap *HeapStruct //最大堆对象
)


//Create 初始化堆对象
func Create(maxSize int32) *HeapStruct {
	hs := new(HeapStruct)
	hs.Size = 0
	hs.Capacity = maxSize
	hs.Elements = make([]int32, hs.Capacity)
	hs.Elements[0] = math.MaxInt32 //哨兵
	return hs
}

```
Insert(插入)
```

//InsertMaxHeap  插入最大堆
func InsertMaxHeap(num int32, maxHeap *HeapStruct) {
	if maxHeap.Capacity == maxHeap.Size {
		panic("最大堆已满")
	}
	maxHeap.Size++
	i := maxHeap.Size
	for ; maxHeap.Elements[i/2] < num; i = i / 2 {
		maxHeap.Elements[i] = maxHeap.Elements[i/2]
	}
	maxHeap.Elements[i] = num
}
```
Delete(删除)
```

//DeleteMaxHeap 最大堆删除，返回删除的元素
func DeleteMaxHeap(maxHeap *HeapStruct) int32 {

	var maxNum, child, parent int32
	if maxHeap.Size == 0 {
		return -1
	}
	maxNum = maxHeap.Elements[1] //待返回的根节点

	tmp := maxHeap.Elements[maxHeap.Size] //待移动的末节点
	maxHeap.Elements[maxHeap.Size] = 0  //将末节点位置置为0表示null
	maxHeap.Size--//节点总数减一
	parent = 1	//待判断父节点为1
	for ; parent*2 < maxHeap.Size; parent = child {
		child = parent * 2
		//取儿子节点中的较大值
		if child != maxHeap.Size && maxHeap.Elements[child] < maxHeap.Elements[child+1] {
			child++
		}
		//判断儿子节点是否大于末节点
		if tmp > maxHeap.Elements[child] {
			break
		}
		//将儿子节点上移
		maxHeap.Elements[parent] = maxHeap.Elements[child]

	}
	//将末节点放入最后的空节点中
	maxHeap.Elements[parent] = tmp
	return maxNum
}

```
BuildHeap(构建堆)  
方法是将N个关键字以任意顺序放入树中，保持结构特性。此时，在从i=N／2开始直到ℹ=1，执行下滤操作，创建一棵具有堆的树。
```

//BuildHeap 建堆
func BuildHeap(nums []int32) *HeapStruct {
	size := len(nums)
	hs := Create(int32(size) + 1)
	//一次将值放入数组中，形成一个完全二叉树
	for i := 1; i <= size; i++ {
		hs.Elements[i] = nums[i-1]
	}
	
	var child, x int32
	for parent := int32(size) / 2; parent > 0; parent-- {
		tmp := hs.Elements[parent]
		child = parent
		x = child
		for {
			child = child * 2
			if child > int32(size) {
				hs.Elements[x] = tmp
				break
			}
			if child != int32(size) && hs.Elements[child] < hs.Elements[child+1] {
				child++
			}
			if tmp > hs.Elements[child] {
				break
			}
			hs.Elements[x] = hs.Elements[child]
			x = child
		}
	}
	return hs
}

```
