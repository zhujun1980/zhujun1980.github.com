---
title: 桶排序和基数排序
layout: post
tags: 算法 排序
category: 算法
comments: true
---
{% include JB/setup %}

桶排序和基数排序都属于**非比较排序**，即它们不比较集合中的元素，而是通过元素本身的定位性来确定次序。

#### 桶排序

桶排序的思想非常简单，有点类似于“收发室”模式：信件按照邮政编码被投入不同的信箱内，然后完成排序。桶排序预先分配若干的“桶”，根据元素的值放入对应的桶中，最后按序扫描每个桶，把桶中的元素收集起来。例如有 __n__ 个取值范围是 __1__ 至 __m__ 的元素，则为它们分配 __m__ 个桶，算法的复杂度大约是O(n+m)：扫描n个元素和m个桶。

虽然桶排序的复杂度可以突破比较排序的**O(nlog<sub>n</sub>)**，但是它需要分配大量的内存空间，比如6位10进制数，就要分配1百万个桶。所以桶排序在工程上的应用并不广泛。

#### 基数排序

桶排序可以很自然的扩展成为基数排序。基数排序扫描元素中每一位，并放入不同的桶中。比如从右至左先取个位数，存入10个桶中，收集桶中数据，完成一次排序；之后再取十位数，以此类推，相比集合的取值范围，每一位的取值范围要小的多，所以基数排序比桶排序要节省很多空间。

####基数排序的实现

	inline int ithdec(unsigned int num, int i) {
		if(i < 0)
			i = -i;

		int base = 1;
		for(int j = 0; j < i - 1; j++)
			base *= 10;

		return (num / base) % 10;
	}

	/** 
	 * example:
	 *
	 *	vector<int> data;
	 *	//insert data
	 * 	radix_sort(data.begin(), data.end(), 10);
	 */
	template<class In>
	void radix_sort(In first, In end, int bl) {
		In ptr;
		std::deque<unsigned int> buckets[10];

		for(int idx = 1; idx <= bl; idx++) {
			for(ptr = first; ptr != end; ptr++)
				buckets[ithdec(*ptr, idx)].push_back(*ptr);
			
			ptr = first;
			for(int i = 0; i < 10; i++) {
				for(auto iter = buckets[i].begin(); 
						iter != buckets[i].end(); 
						iter++, ptr++)
					*ptr = *iter;

				buckets[i].erase(buckets[i].begin(), 
								 buckets[i].end());
			}
		}

		for(ptr = first; ptr != end; ptr++) {
			std::cout<< *ptr<< std::endl;
		}
	}

实现描述

bl是最大位数，比如6位数就传整数6；first和end是数组的头端和尾端。首先，对于每一个数，从右至左扫描，根据该位的数值放入相应的桶中，因为是10进制，所以需要10个桶，每个桶中维护一个队列；每当完成一位的扫描之后，就从0号桶开始收集数据，放回数组中；然后再进行下一位的扫描，直到完成所有位的操作，最后得到排序的数组。

函数 __ithdec(unsigned int num, int i)__ 是计算并返回十进制数num的第i位。

#### 算法特点

1. 非比较排序
2. 非原地置换排序，需要额外的临时存储空间
3. 对于元素的特性有假设，比如需要了解元素的取值范围等
4. 稳定性，即每一次排序后，相同大小的元素次序不变

