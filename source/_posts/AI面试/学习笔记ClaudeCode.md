---
title: claudeCode
tags:
  - 面试
  - claude
categories:
  - 面试
date: 2026-03-16 22:48:00
typora-root-url: ../..
---

claudeCode使用说明

https://www.bilibili.com/video/BV14rzQB9EJj/?spm_id_from=333.1387.upload.video_card.click&vd_source=4c072cbfab3837f2195bf6105cebc175

<!--more-->

# 三种模式

- 默认模式：修改文件前一定询问用户。
- 自动修改模式：自动修改文件。
- plan模式：只讨论、不修改文件。适合构思。

```shell
# 切换模式
shift tab
# 选择回滚点
esc esc 
# 查看任务
/tasks x结束任务
# 压缩上下文内容
/compact
# 清空上下文内容
/clear
# 生成项目级Memory ./CLAUDE.md，（用户级Memory在~/.claude/CLAUDE.md）
/init
# hook
/hook
# subAgent
/agents
```

# Skill和subAgent的区别

- skill：共享主会话的上下文。
- subAgent：独立上下文。

