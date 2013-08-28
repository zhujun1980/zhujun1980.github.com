---
title: 快速排序
layout: post
tags: 算法 排序
category: 算法
comments: true
---
{% include JB/setup %}

####快速排序
对于序列A中的元素A<sub>i</sub>，使得A中小于A<sub>i</sub>的元素都在它的左边，大于A<sub>i</sub>的元素都在它的右边，然后在分别对于左边和右边分别进行快速排序。

一般取序列左边的元素作为每一次排序的A<sub>i</sub>，用两个指针分别从左至右和从右至左遍历序列，交换符合要求的元素（左边的大于A<sub>i</sub>的元素和右边的小于A<sub>i</sub>的元素），快速排序不需要额外的临时存储，所以是__原地置换排序__。

####分治思想
快速排序也体现了分治的思想，只不过它和归并排序的区别是，快速排序的主要工作在于__分__，而__治__的部分基本上没有什么工作。

在每一次划分问题之后：

* 元素A<sub>i</sub>找到了最终的位置
* A<sub>i</sub>的左边、右边这两部分也都在最终的位置区间上


####实现

	template<class In>
	In partition(In first, In end) {
		In L = first + 1;
		In R = end - 1;

		while(L <= R) {
			while(*L <= *first && L != end) L++;
			while(*R > *first && R > first) R--;
			if(L < R)
				swap(*L, *R);
		}
		if(first < R)
			swap(*R, *first);
		return R;
	}

	template<class In>
	void quick_sort(In first, In end) {
		In mid;

		if(first < end - 1) {
			mid = partition(first, end);
			quick_sort(first, mid);
			quick_sort(mid + 1, end);
		}
	}

