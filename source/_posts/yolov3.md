---
title: 模拟面试问答之经典网络模型六：YOLOv3
date: 2020-07-18 21:42:06
tags: 	
	- 深度学习
	- 目标检测
	- 经典网络
categories:
	- 面试
mathjax: true
---

本文总结了面试过程中可能问到的关于YOLOv3模型的一些问题。

[论文地址：YOLOv3: An Incremental Improvement](https://arxiv.org/abs/1804.02767)
![](1.png)

<center><b>图1 Darknet-53网络结构图</b></center>

<!--more-->

# YOLOv3在有哪些主要改进？

1. Darknet-53代替Darknet-19，使用了残差模块，步长为2的卷积层代替池化层进行下采样。

   ![](3.jpg)

   <center><b>图2 YOLOv3网络结构图</b></center>

2. 采用了多尺度结构，在获得32倍下采样特征图后，又开始对特征图进行上采样，并与前面的16倍下采样特征图连接，然后继续对特征图进行上采样，并与前面的8倍下采样特征图连接。分别在三个尺度（8,16,32）的特征图上进行检测，输出分别为13×13×3×(4+1+80)，26×26×3×(4+1+80)，52×52×3×(4+1+80)。

3. 使用独立的logistic分类器（sigmoid）代替softmax分类器进行分类。

# YOLOv3在anchor boxes方面有什么改进？

由于YOLOv3采用了多尺度预测，anchor boxes的设计也做了相应改进。同样是使用k-means进行聚类，YOLOv3聚类了3个尺度，每个尺度聚类了3种尺寸的anchor boxes。

# YOLOv3的损失函数有哪些改进？

置信度和类别用交叉熵损失替换了平方和损失。为了提高小物体的损失贡献，位置回归损失会乘以系数(2-w×h)。

![](2.png)

# 参考资料

https://blog.csdn.net/leonardohaig/article/details/90346325