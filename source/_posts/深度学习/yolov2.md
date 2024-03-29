---
title: 模拟面试问答之经典网络模型五：YOLOv2
date: 2020-07-18 21:42:01
tags: 	
	- 深度学习
	- 目标检测
	- 经典网络
categories:
	- 面试
mathjax: true
typora-root-url: ../..
---

本文总结了面试过程中可能问到的关于YOLOv2模型的一些问题。

[论文地址：YOLO9000: Better, Faster, Stronger](https://arxiv.org/abs/1612.08242)
![](./images/yolov2/1.png)

<center><b>图1 YOLOv2与YOLOv1技巧对比图</b></center>

<!--more-->

# YOLOv2采用了哪些技巧？

1. BN层代替Dropout层。
2. 高分辨率的预训练：预训练时，先使用224×224的输入尺寸训练160个epochs，再使用448×448的输入尺寸训练10个epochs。
3. 使用anchor boxes预测边界框：输入尺寸调整为416×416，输出特征图尺寸为13×13(32倍下采样），准确率下降但是召回率提高。
4. K-means聚类生成anchor box尺寸：聚类的距离函数为$d(box,centroid)=1-IoU(box,centroid)$，若采用欧氏距离会使得尺寸大的框误差也大。实验表明k=5时，在召回率和模型复杂的直接平衡性较好。
5. 直接位置预测：预测相对锚框所属格子左上角坐标的偏移量$(b_x,b_y)$，以及与anchor box宽高的e的指数比例$(b_w,b_h)$，比RCNN系列采用的位置预测方法更易学习。
6. 细粒度特征：类似ResNet，将最后一个26×26的特征图连接到最后一个13×13的特征图上。具体地，首先通过1×1卷积将26×26×512减低通道数为26×26×64，再拆分成4个13×13×64并拼接为13×13×256，然后与13×13×1024进行连接生成13×13×(256+1024)的特征图。
7. 多尺度训练：每隔10个batch随机从{320,352,384,...,608}中选择一个尺度作为输入图像的尺寸。

# YOLOv2的网络结构有哪些改进？

![2](./images/yolov2/2.png)

<center><b>图2 Darknet19网络结构图</b></center>

1. 预训练时，如图2所示，Darknet 19包含19个卷积层（YOLOv1网络包含24个卷积层和2个全连接层）。

   ![3](./images/yolov2/3.png)

   <center><b>图2 YOLOv2网络结构图</b></center>

2. 用于检测时，如图3所示，首先添加三个3×3的卷积，然后输出特征图与细粒度特征中提到的跳跃连接进行拼接，再通过一个3×3卷积调整通道数，最后通过一个1×1卷积输出13×13×125的检测结果，对于13×13个格子，每个格子输出5×25维向量，其中5表示每个格子有五个anchor boxes，25包括20个类别概率，4个位置预测和1个置信度。

3. 预训练时，先使用224×224的输入尺寸训练160个epochs，再使用448×448的输入尺寸训练10个epochs。

# YOLOv2的样本是如何设置的？

类似于YOLOv1，对于一个ground-truth，包含其中心点的格子的5个anchor boxes负责预测该物体框，只有与ground-truth的IoU最大的anchor box会用于计算loss。YOLOv2不再对宽高取根号。与ground-truth的IoU最大或大于0.6的anchor box为正样本。

# 参考资料

https://blog.csdn.net/zijin0802034/article/details/77097894

https://zhuanlan.zhihu.com/p/35325884