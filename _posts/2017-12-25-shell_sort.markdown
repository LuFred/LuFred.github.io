---
layout: post
title: "希尔排序"
subtitle: ""
date: 2017-12-25
author: LuJiangBo
tags: 排序
keyword: 排序,算法
finished: true
---

# 算法
根据维基百科的介绍，希尔排序，也称递减增量排序算法，它是插入排序的一种改进版本,非稳定排序算法。
和插入排序相比，它通过比较相距一定间距的元素来工作，各趟比较所用的距离随着算法的进行而减小，直到只比较相邻元素的最后一趟序列为止。

# 算法原理
基于插入排序我们得出的2个结论：
* 插入排序每次只能将数据移动一位  
* 对于已经排序好的数据操作，效率总是很高    

所以，希尔排序就是基于上面2点问题作出优化。  
首先，希尔排序通过将比较的全部元素分为几个区域来提升插入排序的性能。这样可以让一个元素可以一次性地朝最终位置前进一大步。  
然后,算法再取越来越小的步长进行排序，算法的最后一步就是普通的插入排序，但是到了这步，需排序的数据几乎是已排好的。  
希尔排序使用一个序列h1,h2,...,ht,叫做增量序列，只要h1=1，那么任何增量序列都是可行。在使用增量hk的一趟排序后，对于每一个i我们有A[i]<=A[i+hk],所有相隔hk的元素都是排序的。
下图是我在网上找的一张希尔排序每趟之后的情况
![希尔排序]({{ post.url| prepend: site.url  }}/content/images/201712/shell_sort_01.png)
# 代码实现
```

import (
	"log"
)

func main() {
	var numbs []int = []int{1, 4, 3, 2, 5, 4, 7, 6, 9, 8, 11}
	ShellSort(numbs)
}
func ShellSort(numbs []int) {
	var incre, i, j int
	l := len(numbs)
	for incre = l / 2; incre > 0; incre /= 2 {
		for i = incre; i < l; i += incre {
			var tmp = numbs[i]
			for j = i; j > 0; j -= incre {
				if tmp < numbs[j-incre] {
					numbs[j] = numbs[j-incre]
				} else {
					break
				}
			}
			numbs[j] = tmp
		}
	}
}
```
# 增量序列的选择
一般情况下,比较合理的增量序列有如下2种选择：
* 希尔增量，即：ht=[N/2]和hk=[hk+1/2] (这里的t或者是k都是h的下标)   
* Hibbard增量，即：1,3,7,...,2^k-1。区别在于相邻的增量没有公因子    

# 分析
希尔排序的运行时间依赖于增量序列的选择，这里我直接给出前辈们分析出的几条定理
* 使用希尔增量时，希尔排序的最坏情况运行时间是N^2.  
* 使用Hibbard增量的希尔排序最坏情况运行时间是N^(3/2)  
下图给出了3种步长序列的最坏时间复杂度
![步长序列]({{ post.url| prepend: site.url  }}/content/images/201712/shell_sort_03.png)