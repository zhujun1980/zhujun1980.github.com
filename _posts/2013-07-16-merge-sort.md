---
title: 归并排序
layout: post
tags: 算法 排序
category: 算法
comments: true
---
{% include JB/setup %}

####序列归并
在介绍归并排序之前，首先考察这个问题：

>如何合并两个有序序列 `A` 和 `B` ？

合并方法：首先把B<sub>1</sub>和A中各元素进行比较，找到B<sub>1</sub>的插入位置；然后从插入处开始找B<sub>2</sub>的位置，因为A和B都是有序序列，所以B<sub>2</sub>不必从A<sub>1</sub>重新比较；以此方法完成B中剩余各数的插入，得到A和B合并后的序列。 

####归并排序

在了解如何归并两个有序序列的方法之后，我们就可以使用这种方法进行排序了，具体思路是把序列分成两个子序列分别排序，最后把排序后的序列归并起来。归并排序中序列被不断的划分，直到划分为仅有2个元素的序列为止，比较它们的大小并排序；然后逐级向上进行归并，直到序列的左半部分和右半部分都归并完成。需要说明的一点是，归并的过程需要临时存储保存归并后的序列，所以归并排序__不是原地置换排序__。

####分治思想
归并排序体现了分治的思想，把问题先__分__为若干个子问题，最后合并结果（__治__）。归并排序的大部分工作不在__分__上面，而是在__治__上。

####实现

	template<class In>
	void merge_sort(In first, In end) {
		if((first + 2) == end) {
			if((*first) > *(end - 1))
				swap(*first, *(end - 1));
			return;
		}
		else if((first + 1) < end) {
			size_t s;
			In mid;

			s = (end - first) / 2;
			mid = first + s;
			merge_sort(first, mid);
			merge_sort(mid, end);

			std::vector<int> result;
			In lastA = first;
			In lastB = mid;

			while(lastA != mid && lastB != end) {
				if(*lastA <= *lastB) {
					result.push_back(*lastA);
					lastA++;
				}
				else {
					result.push_back(*lastB);
					lastB++;
				}
			}

			while(lastA != mid)
				result.push_back(*lastA++);
			while(lastB != end)
				result.push_back(*lastB++);

			In ptr = first;
			for(auto ret = result.begin(); 
				ret != result.end(); 
				ret++, ptr++)
				*ptr = *ret;
		}
	}

