---
title: 模拟面试问答之经典网络模型一：RCNN
date: 2020-06-13 18:55:11
tags:
  - 深度学习
  - 目标检测
  - 经典网络
categories:
  - 面试
mathjax: true
---

本文总结了面试过程中可能问到的关于RCNN模型的一些问题。

[论文地址：Rich Feature Hierarchies for Accurate Object Detection and Semantic Segmentation](https://arxiv.org/abs/1311.2524)
![RCNN网络结构](1.png)
<center><b>图1 RCNN网络结构图</b></center>

<!--more-->

# 简述一下RCNN测试时的检测流程？
1. 使用Selective Search在输入图像上提取约2000个候选区域。
2. 缩放每个候选区域到固定尺寸大小，227×227。
3. 将每个候选区域分别输入CNN网络提取特征。
4. 使用二分类SVM分别对每个候选区域进行分类。
5. 对每个类别进行NMS（非极大抑制）。
6. 使用线性回归器分别修正每个候选区域的位置。

# RCNN是如何对候选区域进行缩放的？
各向异性缩放且padding=16。

# 分别介绍一下各向异性和各项同性的缩放？
1. 各向异性直接把图像的宽高拉伸到目标尺寸。
2. 各项同性先按原图像比例进行缩放，然后在再填充到目标尺寸。

# RCNN的CNN网络是如何训练的？
![AlexNet网络结构](2.jpg)
<center><b>图2 AlexNet网络结构图</b></center>

1. 使用AlexNet(网络结构如**图2**所示)或其他主干网络（如VGG16），先在ILSVRC2012的图像分类数据集上进行预训练，最后一层输出维度为1000。
2. 将AlexNet最后的1000维Softmax层替换为随机初始化的20+1维Softmax层，其余层使用预训练得到的参数进行初始化，在PASCAL VOC数据集上对预训练后的AlexNet进行fine-tuning（微调）。
3. Fine-tuning时，定义与ground-truth的IoU大于0.5的候选区域为正样本，否则为负样本（即背景样本），在每次迭代时，采样32个正样本和96个负样本组成一个128的mini-batch。

# RCNN的SVM分类器是如何训练的？
1. 共训练20个二分类SVM。
2. 使用fine-tuning训练后且去掉最后softmax层的AlexNet提取候选区域的特征作为SVM的输入。
3. 定义IoU小于0.3的候选区域为负样本，并进行难负样本挖掘，完整包含物体（IoU=1）的候选区域为正样本，其余样本舍弃。

# RCNN的线性回归器是如何训练的？
1. 共训练20×4个回归器，正则项系数$\lambda$=10000。
2. 输入为AlexNet的第一个全连接层的4096维输出（这里有点问题，论文里面说的是Pool 5，而我理解的Pool 5的输出时6×6×256的），输出为中心点坐标的平移量和宽高的缩放值。
3. 定义IoU大于0.6的样本作为正样本用于训练。

# Fine-tuning后的AlexNet的输出为21维，已经对各个候选区域进行了分类，为什么还要使用SVM进行分类呢？（或为什么不在AlexNet最后使用Softmax进行分类？）
1. 对AlexNet进行fine-tuning时采用的IoU阈值较小，被分为正样本的候选区域可能只包含部分物体，分类精度较低。
2. SVM训练时定义完全包含物体的候选区域才是正样本，分类精度较高。因此，使用SVM能够提高分类精度。

# 为什么fine-tuning训练时和SVM训练时的IoU阈值不一样？
1. Fine-tuning训练时，若IoU阈值设置过高，则可用于的训练样本较少（正样本减少，负样本为正样本的三倍），并且此时AlexNet已进行预训练，易由于样本过少而产生过拟合。因此，设置0.5的IoU阈值只是为了增加可用于训练的样本数量，避免过拟合。  
2. SVM训练时，为了达到进一步提高分类精度的目的，需要设置更为严格的正负样本分类规则。

# 回归器是如何进行边界框回归的？
## 简答
通过预测候选区域中心点坐标的平移量$\Delta x$，$\Delta y$和宽高的缩放值$S_w$，$S_h$来回归边界框。因此对于每个类别，需要4个回归器分别回归$\Delta x$，$\Delta y$，$S_w$，$S_h$，整个模型就包括20×4个回归器。

## 详细
定义$P=(P_x,P_y,P_w,P_h)$表示候选区域的中心坐标和宽高，$G=(G_x,G_y,G_w,G_h)$表示ground-truth的中心坐标和宽高。预测$d_\*(P)$（$\*$表示$x$，$y$，$w$，$h$，下同）得到平移量$\Delta x=P_wd_x(P)$，$\Delta y=P_hd_y(P)$和缩放值$S_w=exp(d_w(P))$，$S_h=exp(d_h(P))$，目标是使变换后的P与G更接近。  
定义$\widehat{G}=(\widehat{G}_x,\widehat{G}_y,\widehat{G}_w,\widehat{G}_h)$表示变换后的$P$，其中：
$$\widehat{G}_x=P_wd_x(P)+P_x$$
$$\widehat{G}_y=P_hd_y(P)+P_y$$
$$\widehat{G}_w=P_wexp(d_w(P))$$
$$\widehat{G}_h=P_hexp(d_h(P))$$
回归器的输入是AlexNet提取的候选区域特征，表示为$\phi(P)$，令$d_\*(P)=w^T_\*\Phi(P)$，$w_\*$就是需要学习的线性变换参数了，可以得到：
$$w^T_x\Phi(P)=d_x(P)=(\widehat{G}_x-P_x)/P_w$$
$$w^T_y\Phi(P)=d_y(P)=(\widehat{G}_y-P_y)/P_h$$
$$w^T_w\Phi(P)=d_w(P)=log(\widehat{G}_w/P_w)$$
$$w^T_h\Phi(P)=d_h(P)=log(\widehat{G}_h/P_h)$$
定义$t_\*$为：
$$t_x=(G_x-P_x)/P_w$$
$$t_y=(G_y-P_y)/P_h$$
$$t_w=log(G_w/P_w)$$
$$t_h=log(G_h/P_h)$$
目标是使P的映射$\widehat{G}$尽量接近$G$，通过以下优化损失函数求解$w_\*$：
$$w_\*=\underset{\widehat{w}_\*}{argmin}\sum(t_\*^i-\widehat{w}_\*^T\phi(P^i))^2+\lambda&#124;&#124;\widehat{w}_\*&#124;&#124;^2$$
可通过最小二乘法或梯度下降求解该问题。由此也可以看出，对于每个类别，需要4个回归器分别回归$\Delta x$，$\Delta y$，$S_w$，$S_h$，整个模型就包括20×4个回归器。

# 回归器为什么不直接预测平移量$\Delta x$和$\Delta y$的值，而是先预测$d_x(P)$和$d_y(P)$，然后分别乘以$P_w$和$P_h$，间接求出$\Delta x$和$\Delta y$？
回归器的输入是候选区域的特征，而每个候选区域在提取特征后的大小时相同的，若是直接预测平移量$\Delta x$和$\Delta y$的值，那么对于两个内容相差无几，但是尺寸不一样的图像，会得到相同平移量，这显然是有问题的。而分别乘候选区域的宽高$P_w$和$P_h$相当于进行归一化，即尺寸大的候选区域预测的平移量大，尺寸小的候选区域的平移量小。

# 回归器预测宽高的缩放值为什么要采用exp？
保证预测值大于0.

# 总结
1. 优点：RCNN引入CNN提取特征，取代手工设计特征，并提升了检测速度；采用了迁移学习的思想，先在大数据集上对模型进行预训练，然后在目标数据集上面进行fine-tuning。
2. 缺点：RCNN使用Selection Search提取2000个候选区域，不够精确；需要对每个候选区域单独提取特征，无法共享特征，且内存占用和耗时较大；训练过程分为三个阶段，繁琐且耗时较大。