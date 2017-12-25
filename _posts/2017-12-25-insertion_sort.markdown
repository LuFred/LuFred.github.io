---
layout: post
title: "插入排序"
subtitle: ""
date: 2017-12-25
author: LuJiangBo
tags: 排序
keyword: 排序,算法
finished: true
---

# 算法
插入排序是众多排序算法中比较简单也比较基础的一种。它是由N-1趟排序组成。即在一个N的数组中，P=1趟到P=N-1趟，插入排序保证从位置0到位置P上的元素是已排序状态。
也就是说插入排序的一个基本要求是：位置0到位置P-1上的元素是已排序的。

# 算法原理
循环遍历数组N，将每一次遍历对象P与前面已经排序好的P-1，P-2···2,1做比较，若每次被比较位置的数值都比P大，这该数值后移一位，最后将P插入空出来的位置。
下图是煤炭后的插入序列
![插入排序]({{ post.url| prepend: site.url  }}/content/images/201712/insert_sort_01.png)

# 代码实现
```
import (
	"log"
)

func main() {
	var numbs []int = []int{1, 4, 3, 2, 5, 4, 7, 6, 9, 8, 11}
	InsertionSort(numbs)
}
func InsertionSort(numbs []int) {
	log.Println("排序前=", numbs)
	var i, j, l int
	l = len(numbs)
	for i = 1; i < l; i++ {
		tmp := numbs[i]
		for j = i; j > 0 && numbs[j-1] > tmp; j-- {
			numbs[j] = numbs[j-1]
		}
		numbs[j] = tmp
	}
	log.Println("排序前=", numbs)
}
```

# 分析
由于嵌套循环的每一个都花费N次迭代，因此插入排序的时间复杂度为O(N^2)。下面讨论为什么时间复杂度为N^2。  
在上面的代码例子中**numbs[j] = numbs[j-1]**这一行的代码在最坏的情况下每次外层循环最多执行i次。所以对所有i的求和,得到总数为  
> ![求和]({{ post.url| prepend: site.url  }}/content/images/201712/insert_sort_02.png)

另一方面,如果输入的数据是是预先已经排序好的,那么内层for循环检测总是立刻判断不成立而终止，也就是说运行时间可达到O(N)。

# 扩展补充
在计算一些简单排序算法的下界时，存在着这样的2条定理。
* N个互异数的数组的平均逆序数是N(N-1)/4
    * 互异数:数值互相不同的2个数
    * 逆序：数组的一个逆序是指数组中具有性质i<j 但是A[i]>A[j]的序偶(A[i],A[j])
* 通过交换相邻元素进行排序的任何算法平均需要Ω(N^2)时间