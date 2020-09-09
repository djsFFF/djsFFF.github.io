---
title: Java项目问题
date: 2020-08-29 20:15:00
tags: 
	- Java
	- 后台开发
categories:
	- 项目
typora-root-url: ..
---

# 论坛后台

## 基于拦截器检查登录状态和权限

![image-20200902164036093](/images/Java%E9%A1%B9%E7%9B%AE%E9%97%AE%E9%A2%98/image-20200902164036093.png)

对于一些需要登录才能访问或者需要一定权限的路径如个人信息，需要使用拦截器检查登录状态。

在用户浏览网页时，需要检查用户登陆状态，通过拦截器实现：

1. 实现HandlerInterceptor接口中的三个方法：

   - preHandler()：在Controller之前执行。获取Cookie中的凭证id，然后在数据库中查询凭证有效状态并根据凭证获取到User对象并保存到ThreadLocal变量hostHolder中。

   - postHandler()：在Controller之后，模板引擎之前执行。从hostHolder获取到User并传给模板引擎。

   - afterCompletion()：在模板引擎之后执行。清除hostHolder。

2. 在实现WebMvcConfigurer接口的类中addInterceptors()配置拦截器：拦截哪些路径，不拦截哪些路径。

## 基于Kafka实现消息队列

私信通知，评论点赞通知等。

### 为什么要用消息队列

1. 不使用消息队列时，用户的请求会直接到达服务器并通过数据库或者缓存响应，而在高并发情况下，可能会超过数据库的承受能力，造成响应速度缓慢甚至宕机。而使用消息队列后，用户的请求数据存入消息队列后就可以立即返回，**减少了响应时间**，消息队列的消费者进程可以异步地处理消息队列中的消息，**降低数据库在高峰期的压力**。
2. 使用消息队列还可以**降低系统的耦合性**，生产者和消费者之间不需要直接通信，而是通过共享消息队列进行交互。

## Redis缓存

## 网站数据统计

### UV（Unique Visitor）

统计不重复IP的访问次数。

### DAU（Daily Active User）

日活跃用户，使用Bitmap，以用户ID作为下标。