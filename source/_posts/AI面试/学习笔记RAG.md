---
title: RAG学习笔记
tags:
  - 面试
  - RAG
categories:
  - 面试
date: 2026-03-16 23:40:00
typora-root-url: ../..
---

RAG学习笔记

https://www.bilibili.com/video/BV1JLN2z4EZQ/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

<!--more-->

RAG（Retrieval-Augmented Generation，检索增强生成）

# 使用场景

## 智能客服

直接把文档丢给大模型会有以下问题

1. context过大
2. 推理成本高
3. 推理速度慢

# 基本流程

准备（提问前）：分片—索引

回答（提问后）：召回—重排—检索

### 分片

把文档分成片段。分词。

### 索引

1. 通过Embedding将片段文本转换为向量。
2. 将片段文本和片段向量存入向量数据库中。

向量：有大小和方向。

Embedding模型：将文本转为向量，存入向量数据库。

### 召回

搜索与用户问题相关的向量。

用户提问—Embedding模型将问题转为向量—向量库匹配top10相似结果

向量相似度

- 余弦相似度：只看方向，不看长度，适合文本语义。
- 欧式距离：看空间直线距离，适合位置、坐标，不适合文本。
- 点积：速度快，但需要做归一化。

### 重排

top10中选取top3。

cross-encoder计算向量相似度：计算成本更高，准确率高。

### 生成

用户问题 + 参考资料 — 大模型 — 归纳总结结果

