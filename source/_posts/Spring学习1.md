---
title: Spring学习1
date: 2020-08-29 20:15:00
tags: 
	- Spring
	- Java
categories:
	- Spring
typora-root-url: ..
---

Spring学习。

<!--more-->

# IoC

IoC（Inverse of Control，反转控制）是一种设计思想，将原本手动创建对象的控制权交给Spring框架来管理。

# AOP

AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，减少系统的重复代码，降低模块的耦合度，有利于提高系统的可扩展性和可维护性。

![image-20200830194807094](/images/Spring%E5%AD%A6%E4%B9%A01/image-20200830194807094.png)

AOP基于动态代理，如果代理的对象实现了某个接口，那么Spring AOP会使用JDK Proxy去创建代理对象，而对于没有实现接口的对象，使用Cglib生成一个被代理对象的子类来作为代理。

## Spring AOP与Aspect AOP

- Spring AOP：运行时增强，基于动态代理，简单。
- Aspect AOP：编译时增强，基于字节码操作，强大，更快。

# Bean的作用域

@scope("singleton")

- **singleton**：在Spring IoC容器中只存在一个Bean实例，即以单例方式存在，默认值。
- **prototype**：每次从容器中调用Bean时，都返回一个新的实例。
- request：每次HTTP请求都会创建一个新的Bean，仅适用于WebApplicationContext环境。
- session：同一个HTTP Session共享一个Bean，仅适用于WebApplicationContext环境。
- global-session：全局session，一般用于Portlet应用环境，仅适用于WebApplicationContext环境。

# Bean的生命周期

![preview](/images/Spring%E5%AD%A6%E4%B9%A01/880f402c83a0e04f2b4ccfcec3239dc8_r.jpg)

1. 实例化Bean：new操作，调用构造方法。

2. 设置对象属性：利用依赖注入调用setter设置属性值。

3. BeanNameAware接口，Spring将Bean的id传给setBeanName()。

4. BeanFactoryAware接口，Spring调用setBeanFactory()，将BeanFactory容器实例传入。

5. ApplicationContextAware接口，Spring调用setApplicationContext()，将bean所在应用的上下文引用传入。

6. BeanPostProcessor接口，Spring调用他们的postProcess**Before**Initialization()。

7. InitializingBean接口，Spring调用他们的afterPropertiesSet()。

8. 调用配置文件中包含init-method属性的方法

9. BeanPostProcessor接口，Spring调用他们的postProcess**After**Initialization()。

10. Bean已经准备就绪，可以被使用。

11. DisposableBean接口，Spring将调用destory()。

12. 调用配置文件中包含destroy-method属性的方法。

# Spring MVC

Spring MVC是一个基于MVC设计模式的框架，通过把Model，View，Controller分离，将表现层（其他两层：业务层，数据访问层）进行解耦，简化开发，减少出错，方便组内人员之间的配合。

## 流程

1. 用户发送请求到前端控制器DispatcherServlet。
2. DispatcherServlet根据请求调用HandlerMapping，获取到对应的Handler。
3. DispatcherServlet调用HandlerAdapter来执行Handler，并返回一个ModelAndView对象。
4. DispatcherServlet调用ViewResolver对视图进行解析，并将Model数据填充到视图中。
5. 最后DispatcherServlet将View返回给用户。

# 相关设计模式

1. 工厂模式：BeanFactory、ApplicationContext创建Bean对象。
2. 代理模式：Spring AOP的实现。
3. 单例模式：Bean的默认作用域。
4. 装饰器模式：不同的客户在每次访问中会根据需求去访问不同的数据库。
5. 观察者模式：Spring事件驱动模型。
6. 适配器模式：Spring AOP的增强或通知、Spring MVC使用适配器模式适配Controller。

# Spring事务

## 管理事务的方式

- 编程式事务：代码中硬编码。
- 声明式事务：在配置文件中配置。基于XML或基于注解。

## 事务传播行为

规定了事务方法或事务方法嵌套调用时的事务传播方式。

```java
@Transactional(propagation=propagation.REQUIRED)
methodA {
	methodB();
}

@Transactional(propagation=propagation.[以下七种传播机制])
methodB {}
```

### required

> 如果当前没有事务，则新建一个事务，如果已经存在一个事务中，则加入该事务。

如果当前没有事务，则创建一个事务；如果当前已经处于一个事务中，则加入到该事务。默认设置。

### supports

>支持当前事务，如果当前没有事务，则以非事务方式执行。

如果当前没有事务，则以非事务方式执行；如果当前已经处于一个事务中，则加入到该事务。

### mandatory

> 使用当前事务，如果当前没有事务，则抛出异常。

如果当前没有事务，则抛出异常；如果当前已经处于一个事务中，则加入到该事务。

### requires_new

> 新建事务，如果当前存在事务，则把当前事务挂起。

如果当前没有事务，则创建一个事务；如果当前已经处于一个事务中，则把已存在的事务挂起，新创建一个事务并执行。

### not_supported

> 以非事务方式执行，如果当前存在事务，则把当前事务挂起。

如果当前没有事务，则以非事务方式执行；如果当前已经处于一个事务中，则把已存在的事务挂起。

### never

> 以非事务方式执行，如果当前存在事务，则抛出异常。

如果当前没有事务，则以非事务方式执行；如果当前已经处于一个事务中，则抛出异常。

### nested

如果当前没有事务，则创建一个事务；如果当前已经处于一个事务中，则创建一个嵌套事务。