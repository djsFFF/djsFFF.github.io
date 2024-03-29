---
title: 模拟面试问答之经典网络模型二：Fast RCNN
date: 2020-07-01 21:35:34
tags:
  - 深度学习
  - 目标检测
  - 经典网络
categories:
  - 面试
mathjax: true
typora-root-url: ../..
---

本文总结了面试过程中可能问到的关于Fast RCNN模型的一些问题。

[论文地址：Fast R-CNN](https://arxiv.org/abs/1504.08083)
![1](./images/FastRCNN/1.jpg)

<center><b>图1 Fast RCNN网络结构图</b></center>

<!--more-->

# 简述一下Fast RCNN测试时的检测流程？
1. 使用Selective Search在输入图像上提取约2000个候选区域。
2. 将原始图像输入卷积网络，获得最后一个池化层之前的特征图（16倍下采样）。
3. 对于每个候选区域，RoI pooling层将候选区域定位到特征图中的对应区域产生RoI，然后将每个RoI转化为相同尺寸（7×7）。
5. 通过全连接层将RoI转化为一维向量，分别输入并行的分类器和回归器。分类器由一个全连接层和Softmax组成，对每一个RoI输出20+1个类别概率。回归器输出通过一个全连接层对每一个RoI输出20×4维向量，即所有类别下回归RoI的位置和大小。

# 描述一下RoI pooling的工作原理？
先将候选区域定位到特征图中的对应区域产生RoI，然后将RoI划分为H×W的区域，然后分别取每个区域的最大值代替该区域，从而将RoI尺寸调整为H×W。

# 两次坐标量化是指什么？
1. 第一次量化时将候选区域映射到特征图上时，需要将候选区域坐标除以16并取整。
2. 第二次量化是指RoI pooling时，需要将RoI划分为H×W个部分，对左上角坐标采用向下取整，对右下角坐标采用向上取整。两次量化操作会引入误差，Mask RCNN中会解决这一问题。

# Fast RCNN是如何训练的？
1. 以VGG16为例，先在ImageNet上训练一个1000类的分类网络。
2. 修改预训练后的网络结构，将最后一个池化层替换为RoI pooling层；将最后一个全连接层替换为并行的分类器和回归器；网络输入替换为图像+候选区域集合。
3. 将修改后的网络在Pascal VOC上进行fine-tuning，定义IoU大于0.5的为正样本，IoU为0.1-0.5之间的为负样本，每次迭代时包括2张图像，每张图像随机采样16个正样本和48个负样本用于训练；采样0.5概率的水平翻转进行数据增强。
4. 采用分类损失和回归损失的联合组成的多任务损失函数进行训练。

# Fast RCNN的损失函数是如何构成的？
## 简答
Fast RCNN采用多任务损失函数，由分类损失和回归损失联合组成。分类采用交叉熵损失函数，回归采用Smooth L1损失函数。

## 详细
Fast RCNN的多任务损失函数如下：
$$L(p,u,t^u,v)=L_{cls}(p,u)+\lambda [u\geq 1]L_{loc}(t^u,v)$$
其中，$L_{cls}(p,u)=-logp_u$表示真实分类u的概率$p_u$的对数损失。$\lambda$用于平衡分类损失和回归损失，通常取1。$[u\geq 1]$为指示函数，$u\geq 1$时取1否则取0，即当p为负样本时忽略回归损失。$L_{loc}(t^u,v)$为四个Smooth L1损失的和，分别是两个平移量和两个缩放值（具体参考上一篇关于RCNN的博客）。Smooth L1损失函数如下所示：
$$Smooth_{L1}(x)=\begin{cases} 0.5x^2 & if&#124;x&#124;<1 \\\\ &#124;x&#124;-0.5 & otherwise \end{cases}$$
Smooth L1损失函数在x较大时（训练初期）采取L1损失函数形式，避免了L2损失在训练初期导数大的问题；在x较小（训练后期）时采取L2损失函数形式，避免了L1损失函数在训练后期导数大的问题。

# 总结
1. 优点：Fast RCNN解决了RCNN的两大问题：①一次性提取特征然后对候选区域进行特征映射，实现特征共享，极大地节约了时间和空间消耗；②将RCNN的三个模型整合为一个模型，一次性进行分类和回归。
2. 缺点：仍然存在与RCNN一样的问题，即使用Selective Search进行候选框提取。

