---
title: HashMap
date: 2020-08-09
tags: 
	- JAVA
	- 后台
categories:
	- JAVA 
typora-root-url: ..
---

HashMap源码解读。

<!--more-->

# put()方法存储过程（JDK8）

1. 调用`put(key, value)`方法，若数组`Node<K, V>[] table`为空或table长度为0，则进行初始化`table = new Node<K, V>[16]`，即数组默认初始化容量为16。（惰性加载）

2. 调用`hash()`方法计算key的`hash`值。

   ```java
   static final int hash(Object key) {    
       int h;    
       // 使用hash值计算下标时，若n比较小，则hash的高16位无法参与运算
       // 为了减少哈希碰撞，需要使高16位也能参与下标计算
       // h ^ (h >>> 16)相当于高16位不变，低16位变为低16位与高16位的异或结果
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   ```

3. 结合`hash`值和容量`n`计算索引节点下标`index`，并获得索引结点`p`。

   ```java
   index = (n - 1) & hash
   p = table[index]
   ```

4. 若索引结点p为null，则创建新结点`p = newNode(hash, key, value, null)`存储插入数据。

   否则从p开始遍历以p为头结点的链表（拉链法）：

   - 若链表中存在结点`e`使得`e.hash == hash && (e.key == key || key.equals(e.key))`，则使用value更新e的value，即`e.value = value`。**存储结束。**
   - 否则在链表尾部创建一个新的结点存储插入数据。

5. 插入新数据后，map容量`size`自增1，若容量大于扩容阈值则调用`resize()`方法进行扩容。

# 扩容

扩容阈值=容量×加载因子(0.75)，因此扩容阈值默认初始值为16*0.75=12。

扩容时，扩容阈值和容量为原来的两倍。

对于HashMap中的每个非根结点e，若`(e.hash & oldCap) == 0`则扩容后的索引要么不变，若`(e.hash & oldCap) == 1`要么为原索引+原容量。根节点重新计算索引`newIndex = (newCap - 1) & hash`。oldCap和newCap为新旧桶的数量。

# 红黑树与链表转换

- 当某个链表长度（桶中结点数）大于8，且桶的数量大于64时，该**链表转为红黑树**。
- 当某个红黑树结点数小于等于6时，该**红黑树转回链表**。

## 为什么转换的阈值分别为8和6？

红黑树的平均查找长度为log(n)，链表的平均查找长度为n。

当n为8时，`log(8) = 3 < 8 / 2`，此时红黑树查找效率更高，空间换时间较划算。

当n为6时，`log(6) = 2.6 < 6 / 2`，此时红黑树与链表查找效率相差不大，时间换空间较划算。

此外，源码中解释根据泊松分布，链表长度大于8的概率极小。

## 为什么桶的数量要大于64

源码中解释，如果存在链表结点数大于8且桶的数量小于等于64，说明桶中的结点过多，应调用`resize()`方法进行扩容：

```java
if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) {
    resize();
}
```

避免扩容和树型化选择的冲突。

# 指定初始化容量n必须为2的次幂

## 为什么？

为了减少hash碰撞，使数据均匀分配。

HashMap计算索引的方式为`index = (n - 1) & hash`，当n为2的次幂时这种索引计算方式与取余的结果一致，即`(n - 1) & hash == hash % n`。

n- 1的形式为若干个连续的0和若干个连续的1如`16 - 1 = 15 = 00000000 00000000 00000000 00001111`。

## 若n不是2的次幂会怎么样？

会向上扩展为2的次幂，如n=10变为16。

具体实现代码：

```java
// 若不执行减一操作，输入cap刚好为2的次幂时，会将容量扩大为2*cap。
int n = n - 1;// n = 10为例,此时n = 9
n |= n >>> 1;   // 9 | 4 = 13
n |= n >>> 2;	// 13 | 3 = 15
n |= n >>> 4;	// 15 | 0 = 15
n |= n >>> 8;	// 15 | 0 = 15
n |= n >>> 16;	// 15 | 0 = 15
return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1; // n + 1 = 16
```

# 加载因子（loadFactor）

加载因子 = 容量 / 桶的数量，可以衡量HashMap的稀疏程度，官方设定为加载因子0.75时，HashMap足够拥挤了，需要进行扩容。

# 初始容量设置

若已知需要插入的数据个数为n，则应该设置`初始容量 = (n / 0.75) + 1`。
