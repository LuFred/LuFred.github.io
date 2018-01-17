---
layout: post
title: "快速排序"
subtitle: ""
date: 2017-12-25
author: LuJiangBo
tags: 算法
keyword: 排序,算法,数据结构
finished: true
---
# 快速排序

快速排序又称划分交换排序，它也是[分治](https://zh.wikipedia.org/wiki/%E5%88%86%E6%B2%BB%E6%B3%95)排序的一种。它的平均运行时间是O(NlogN).

## 算法原理

利用分治法将一个待排序序列分为2个子序列，递归进行直到每个序列的元素为1，再向上合并已排好的序列。  
步骤：  
* 从数列中随机挑出一个元素，称为"基准"（pivot）
* 重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（相同的数可以到任何一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
* 递归地把小于基准值元素的子数列和大于基准值元素的子数列排序。

## go语言代码实现

```

package main

import "log"

func main() {
	var arr = []int{1, 7, 3, 5, 0, 11, 65, 23, 55, 1, 23, 21, 65}
	l := len(arr)
	log.Println("原始序列：", arr)
	QuickSort(arr, 0, l-1)
	log.Println("排序后：", arr)
}

//QuickSort 排序函数
//s:起始位
//e:结束位
func QuickSort(arr []int, s, e int) {
	if s >= e {
		return
	}
	ck := arr[s]
	l := s
	r := e
	for l < r {
		for arr[r] > ck { //从右到左滑动，直到遇到第一个比基准点小的元素
			r--
		}
		Swep(arr, l, r)             //交换元素位置
		for l < r && arr[l] <= ck { //从左到右滑动，直到遇到第一个比基准点大的元素
			l++
		}
		Swep(arr, l, r)
		if r > s {
			r--
		}
	}
	//此时比基准点大的元素都在基准点右边
	//比基准点小的元素都在基准点左边
	QuickSort(arr, s, r)   //将基准点小的元素再执行快排
	QuickSort(arr, r+1, e) //将基准点大的元素再执行快排
}

//Swep 元素位置交换
func Swep(arr []int, x, y int) {
	if x == y {
		return
	}
	arr[y] = arr[x] + arr[y]
	arr[x] = arr[y] - arr[x]
	arr[y] = arr[y] - arr[x]
}

```


## 分析

快速排序也是递归排序，多变点在于基准点的选取，在上面的例子中始终用序列的第一个元素作为基准点，这在排序大部分已经排序好的序列时很可能会使时间复杂度上升到N^2.  
因此一个好的基准点选取方法在快速排序中也很重要。