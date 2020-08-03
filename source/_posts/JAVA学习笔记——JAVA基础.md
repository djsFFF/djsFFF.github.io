---
title: JAVA学习笔记——JAVA基础
date: 2020-07-28 17:48:12
tags:
	- JAVA
categories:
	- JAVA
---

记录《Java核心技术卷一》学习过程中的一些笔记。

<!--more-->

# Java的基本程序设计结构

1. public访问修饰符。

2. 数据类型
| 类型     |         | 存储需求 |备注|
| -------- | ------- | -------- |--|
| 整形     | int     | 4字节    |
|          | short   | 2字节    |
|          | long    | 8字节    |
|          | byte    | 1字节    |
| 浮点类型 | float   | 4字节    |6-7位精度|
|          | double  | 8字节    |15-16位精度|
| 字符     | char    | 2字节    |
| 布尔类型 | boolean | 4字节    |

# 对象与类

1. `final`关键字表示变量只能被赋值一次，赋值后无法更改。类中的final字段必须在构造对象时初始化。
2. 被`static`修饰的字段为类字段，所有实例共享。非静态字段则每个实例都有自己的一个副本。

# 继承

1. 包装器、装箱、拆箱

   - 包装器：Integer、Long、Float、Double、Short、Byte、Character、Boolean

   - 自动装箱、拆箱（编译器的工作）：

     ```
     List<Integer> list = new ArrayList<>();
     list.add(3); # 自动装箱，自动变换为list.add(Integer.valueOf(3));
     int n = list.get(0); # 自动拆箱，自动变换为list.get(0).intValue();
     ```


# 接口

1. 接口的属性：
   - 接口中所有方法都是public的。
   - 不能new实例化一个接口；可以声明一个接口变量，接口变量只能引用实例化了这个接口的类对象。
   - 接口可以继承自接口，每个类只能有一个超类，但可以实现多个接口。
   - 接口中不能有实例字段，但是可以包含常量`public static final`。
2. 类优先：一个类继承了一个超类，实现了一个接口，并从超类和接口中继承了相同的方法，此时只考虑超类中的方法。

# 异常

Error和Exception都是继承自Throwable。

- Error：系统的内部错误和资源耗尽等错误。
- Exception：
  - IOException：IO错误导致的异常。
  - RuntimeException：编程错误导致的异常。

派生与Error类和RuntimeException类的异常为非检查型异常，其他异常为检查型异常。

# 线程

具有6种状态，可通过getState方法查看：

- New（新建）：new操作新建一个新线程时，还没开始运行。
- Runnable（可运行）：调用start()方法，可能正在运行也可能没运行。
- Blocked（阻塞）：线程试图获得一个对象锁，而这个锁目前被其他线程占有，该线程就会被阻塞。
- Waiting（等待）：线程等待另一个线程通知调度器。
- Time waiting（计时等待）：Thread.sleep，Object.wait，Thread.join，Lock.tryLock和Condition.await。
- Terminated（终止）：run方法正常退出或意外终止。

