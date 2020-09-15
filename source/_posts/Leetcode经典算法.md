---
title: Leetcode经典算法
date: 2020-09-13 22:15:27
tags: 
	- 算法
categories:
	- 算法
---

记录Leetcode中经典的算法思路。

<!--more-->

# 两数之和

![image-20200913175716328](../images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20200913175716328.png)

**思路：**

- 排序+双指针，时间O(nlogn)，空间O(1)。
- HashMap，一次遍历检查HashMap是否已有target - nums[i]，否则添加当前元素到HashMap中。

# 三数之和

![image-20200913180029903](../images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20200913180029903.png)

**思路：**排序+三指针，遍历排序后nums数组，对于每一个元素检查是否该元素后面是否存在l，r使得nums[i] + nums[l] + nums[r] = 0。