---
title: RAG学习笔记
tags:
  - 面试
  - MCP
categories:
  - 面试
date: 2026-03-16 23:40:00
typora-root-url: ../..
---

MCP学习笔记

https://www.bilibili.com/video/BV1uronYREWR/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

https://www.bilibili.com/video/BV1Y854zmEg9/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

https://www.bilibili.com/video/BV1v9V5zSEHA?spm_id_from=333.788.player.switch&vd_source=4c072cbfab3837f2195bf6105cebc175

<!--more-->

MCP（Model Context protocol，模型上下文协议）

![image-20260317220515792](./images/RAG学习笔记/image-20260317220515792.png)

MCP Server调用方式

- uvx：uv tool run xxx
- npx：

Cline在系统提示词中限制大模型调用工具的规范化格式：如xml格式

也会告诉模型调用MCP的输入限制：JsonSchema

![image-20260318001021446](./images/学习笔记MCP/image-20260318001021446.png)
