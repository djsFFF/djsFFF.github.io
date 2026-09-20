---
title: 学习笔记contextEngineering
tags:
  - 面试
  - contextEngineering
categories:
  - 面试
date: 2026-03-17 12:48:00
typora-root-url: ../..
---

Context Engineering

https://www.bilibili.com/video/BV11zbPzVEUe/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

<!--more-->

# ContextWindow

模型输入最多的Token数量

1token = 0.75个英文=1.5个汉字

- Context window有限
- 输入太杂乱会影响模型理解
- 输入越多，成本越高

# ContextEngineering

设计模型的输入内容

### 保存Context

- 信息持久化，保存在内存/硬盘

### 选择Context

- 静态选择：核心配置文件必须选择，比如CLAUDE.md
- 动态选择：挑选相关memory、RAG、工具

### 压缩Context

- 压缩总结历史消息

### 隔离Context

- 多Agent隔离Context