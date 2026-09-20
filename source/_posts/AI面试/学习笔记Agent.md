---
title: claudeCode
tags:
  - 面试
  - Agent
categories:
  - 面试
date: 2026-03-17 12:48:00
typora-root-url: ../..
---

Agent概念、原理、构建模式

https://www.bilibili.com/video/BV1TSg7zuEqR/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

<!--more-->

# 运行模式

## ReAct模式

最广泛的模式

提交任务—思考—是否调用工具—行动（调用工具）—观察（查看工具结果）—思考（是否继续调用工具）—...—最终答案

![image-20260317125212538](./images/Agent学习笔记/image-20260317125212538.png)

系统提示词

![image-20260317125504459](./images/Agent学习笔记/image-20260317125504459.png)

![image-20260317130423723](./images/Agent学习笔记/image-20260317130423723.png)

## Plan-And-Execute模式

先规划再执行

Plan模型：给出执行计划。

Re-Plan模型：给出新的执行计划。

执行Agent：可以是ReAct模式Agent进行执行。

![image-20260317130607639](./images/Agent学习笔记/image-20260317130607639.png)

![image-20260317130946057](./images/Agent学习笔记/image-20260317130946057.png)