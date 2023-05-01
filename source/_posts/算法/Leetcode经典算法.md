---
title: Leetcode经典算法
date: 2020-09-13 22:15:27
tags: 
	- 算法
categories:
	- 算法
typora-root-url: ../..
---

记录Leetcode中经典的算法思路。

<!--more-->

# 两数之和1

![image-20200913175716328](./images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20200913175716328.png)

**思路：**

- 排序+双指针，时间O(nlogn)，空间O(1)。
- HashMap，一次遍历检查HashMap是否已有target - nums[i]，否则添加当前元素到HashMap中。

# 三数之和15

![image-20200913180029903](./images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20200913180029903.png)

**思路：**排序+三指针，遍历排序后nums数组，对于每一个元素检查是否该元素后面是否存在l，r使得nums[i] + nums[l] + nums[r] = 0。

# 最长连续序列128

![image-20201021221252211](./images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20201021221252211.png)

**思路：**将数组元素存入set，然后再次遍历数组，如果set中存在比当前值小的数，则跳过，否则循环检查是否连续值。

# 和为k的子数组560（437）

![image-20201022155443181](./images/Leetcode%E7%BB%8F%E5%85%B8%E7%AE%97%E6%B3%95/image-20201022155443181.png)

**思路：**前缀和+哈希表。遍历数组，记录当前遍历过的元素的和，将其出现次数存放在HashMap中。对于每个元素检查HashMap中是否存在前缀和为sum-k，若存在则将结果加上该前缀和出现的次数。