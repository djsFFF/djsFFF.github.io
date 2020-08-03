---
title: 模拟面试问答之经典网络模型三：Faster RCNN
date: 2020-07-15 09:03:28
tags:
  - 深度学习
  - 目标检测
  - 经典网络
categories:
    - 面试
mathjax: true
---

本文总结了面试过程中可能问到的关于Faster RCNN模型的一些问题。

[论文地址：Faster R-CNN](https://arxiv.org/abs/1506.01497)
![Faster RCNN网络结构](1.png)
<center><b>图1 Faster RCNN网络结构图</b></center>

<!--more-->

# 简述一下Fast RCNN测试时的检测流程？
1. 调整输入图像尺寸为16的倍数。
2. 使用CNN（以VGG16为例）提取特征图。
3. RPN（Region Proposal Network）在特征图上生成一系列anchor box，并通过两个分支分别判断anchor box是否包含物体以及粗略位置修正。
4. 将特征图和anchor boxes一起输入到RoI pooling层，与Fast RCNN一样，对于每个anchor boxes，RoI pooling层将候选区域定位到特征图中的对应区域产生RoI，然后将每个RoI转化为相同尺寸（7×7）。
6. 经过两个全连接+ReLU层将RoI转化为一维向量，分别输入并行的分类器和回归器。分类器由一个全连接层和Softmax组成，对每一个RoI输出80+1个类别概率。回归器输出通过一个全连接层对每一个RoI输出80×4维向量，即所有类别下回归RoI的位置和大小。

# Faster RCNN相对于Fast RCNN的主要改进在什么地方？
1. Faster RCNN不再使用Selective Search提取候选区域，而是采用卷积网络（RPN）产生候选框。
2. RPN网络在生成候选框后会对候选框进行筛选，只保留可能包含物体的候选框（～300个），为后续计算节约计算成本。

# 介绍一下RPN网络？
![Faster RCNN网络结构](3.png)
<center><b>图2 RPN网络结构图</b></center>

1. 首先通过一个3×3的卷积进一步集中特征信息，然后在特征图上的每个特征点生成9个anchor box（三个尺度，三个比例），把生成的anchor boxes分别送入分类分支和回归分支。
2. 对于分类分支，通过一个1×1卷积+Softmax输出每个anchor box存在物体的概率。
3. 对于回归分支，通过一个1×1卷积输出每个anchor box的粗略位置估计，包括2个平移量（$\Delta x$，$\Delta y$）和两个缩放量($S_h$，$S_w$)，具体参考前面关RCNN的博客。
4. 最后Proposal层根据分类和回归分支结果筛选候选框。首先根据回归结果修正候选框位置，然后根据分类结果筛选出前k（6000）个存在物体概率最高的候选框，并去除尺寸较小的和超出边界的候选框，再使用NMS剔除重叠候选框（0.7阈值），最后将筛选结果（约300个）送入RoI pooling层。

# Faster RCNN是如何训练的？
1. 交替训练（论文采用）：Faster RCNN可以看做时Fast RCNN + RPN，可以将Fast RCNN和RPN单独进行训练，但这两部分共享VGG16部分。  
① 首先在ImageNet上面预训练VGG16；  
② 使用①中的VGG16参数结合RPN进行训练；  
③ 使用①中的VGG16参数组成Fast RCNN，并利用训练后的RPN网络生成候选框来训练Fast RCNN；
④ 使用③中的VGG16参数并结合RPN进行训练，VGG16参数不更新。  
⑤ 使用③中的VGG16参数Fast RCNN，并利用④中的RPN网络生成候选框来训练Fast RCNN，VGG16参数不更新。
2. 近似联合训练：将Faster RCNN整体进行训练，包括四个损失函数，RPN二分类损失，RPN回归损失，最后的多分类损失和回归损失。

# Faster RCNN的损失函数是如何构成的？
1. 对于RPN训练，定义IoU最大或者IoU大于0.7的候选框为正样本，IoU小于0.3的候选框为负样本，其余候选框舍弃，每张图片随机采样128个正样本和128个负样本用于训练，若正样本过少，则增加负样本，保持样本总数为256。RPN的多任务损失函数为：
$$L({p_i},{t_i})=\frac {1}{N_{cls}}\sum_i L_{cls}(p_i,p_i^\*)+\lambda \frac {1}{N_{reg}}\sum_i p_i^\*L_{reg}(t_i,t_i^\*)$$
其中，对于正样本$p_i^\*=1$，否则为0。$L({p_i},{t_i})=-logp_i$表示第i个候选框含有物体的概率$p_i$的对数损失。$L_{loc}(t^u,v)$为四个Smooth L1损失的和，分别是两个平移量和两个缩放值（关于Smooth L1损失可参考上一篇关于Fast RCNN的博客）。
2. 对于网络末端的分类器和回归器的训练，采用与Fast RCNN相同的损失函数。
3. 实际上RPN的损失函数与Fast RCNN唯一的区别就在于一个是二分类，一个是多分类。此外，两者对于正负样本的划分有所区别。

# 总结
1. 优点：Faster RCNN利用RPN生成候选框，舍弃了耗时的Selective Search方法，无论是检测速度和精度都得到了提升。
2. 缺点：Faster RCNN的两阶段方法无法达到实时检测的效果。