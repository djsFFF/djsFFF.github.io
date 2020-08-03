---
title: JAVA面试问题
date: 2020-07-26 18:25:46
tags:
	- JAVA
	- 面试
categories:
	- 面试
---

# JAVA基础

本文总结了面试过程中可能问到的关于JAVA的一些问题。

<!--more-->

## 面向对象的三大特性

- 封装：将对象的实现细节隐藏起来，通过公共的方法向外暴露该对象的功能。
- 继承：子类通过继承可以直接或间接地获得父类的成员。继承破坏了封装，因为父类的实现细节对子类是透明的。
- 多态：同一个行为具有多个不同表现形式的能力。条件：继承、重写、向上转型。

## 重写和重载

- 重写：子类对父类允许访问的方法实现过程并重新编写，返回值和形参不能改变。
- 重载：在一个类里面，方法名字相同，参数不同，返回类型可以相同也可以不同。

## 抽象类和接口的区别

- 抽象类中可以没有抽象方法，接口中的方法必须是抽象方法。
- 抽象类中可以有普通成员变量，接口中的变量必须是static final类型，必须初始化，只有常亮，没有变量。
- 抽象类只能单继承，接口可以继承多个父接口。

## 对象初始化顺序

1. 调用父类的构造函数。
2. 按代码顺序执行静态成员变量的初始化。
3. 调用自身构造函数。

## StringBuffer和StringBuilder

- StringBuffer线程安全，StringBuilder线程不安全。

- String的+实际上是通过StringBuilder实现的。

## Exception和Error

- 都继承自Throwable
- Exception分为CheckedException和UnCheckedException：CheckedException必须显式捕获，比如IO操作；UnCheckedException不用显式捕获，比如空指针、数组越界等。

# JAVA集合

## 常用集合

Map和Collection是所有集合框架的父接口。

- Map：HashMap、ConcurrentHashMap、HashTable和Properties等。
- Collection：
  - Set：HashSet、TreeSet和LinkedHashSet等，元素有序且可以重复。
  - List：ArrayList、LinkedList、Stack和Vector等，元素无序且不可以重复。

## HashMap、HashTable和ConcurrentHashMap

- HashMap基于数组（主体）+链表（解决哈希冲突）/红黑树（存储）实现，允许key/value为空值，线程不安全，不能保持插入顺序。
- HashTable不允许key/value为空值，使用了synchronized关键字，线程安全。
- ConcurrentHashMap可以使用空值作为key/value，线程安全，HashTable每次同步执行时要锁住整个结构，ConcurrentHashMap将Hash表分为16个桶，每次操作只锁住需要用到的桶。

## ArrayList和Vector

- ArrayList是线程不安全的，在数据量达到容量一半时，增长0.5倍容量+1的空间。
- Vector加了synchronized关键字，是线程安全的，增长1倍容量的空间。

## ArrayList和LinkedList

- ArrayList基于数组实现，查找快，删减慢。
- LinkedList基于双向链表实现，查找慢，删减快。且封装了队列和栈的调用。

# JAVA线程同步

## volatile关键字

- 用于修饰可能被多线程同时访问的变量。
- 相当于轻量级synchronized，volatile能保证有序性（禁用指令重排序）、可见性。
- 被volatile修饰的变量改变后会立即同步到主线程中，保持变量的可见性。

## 单例模式双重校验锁为什么要使用volatile关键字?

假设某个线程检测到单例对象为空并开始实例化对象，步骤包括分配内存空间、初始化对象和将对象指向内存空间。编译器可能由于性能的原因将先指向内存空间再进行初始化，在这种情况下，可能会有另外一个线程在单例初始化之前检测到单例不为空（已指向内存），并进行访问，导致程序崩溃。而volatile能够防止指令重排序，避免这一问题。

## wait和sleep

- sleep是Thread的静态方法，可以在任何地方调用，不会释放锁。

- wait是Object的成员方法，只能在synchronized代码块中调用，会释放锁。

## 锁池和等待池

- 某个对象的锁已被一个线程拥有，其他线程要执行该对象的synchronized方法获取锁时就会进入该对象的**锁池**，锁池中的线程会去竞争该对象的锁。
- 某个线程调用了某个对象的wait方法，该线程就会释放该对象的锁，进入该对象的**等待池**，等待池中的线程不会去竞争该对象的锁。
- 调用notify会随机唤醒等待池中的一个线程进入锁池。
- 调用notifyAll会唤醒等待池中的所有线程进入锁池。

## synchronized原理

进入时，执行monitorenter，将计数器+1，释放锁时，执行monitorexit，计数器-1。计数器为0时表示锁空闲，可以占用，否则等待。

## synchronized和lock

- synchronized是JAVA关键字，会自动释放锁，无法中断等待锁。
- lock是一个接口，通常在finally中手动释放锁，可以中断等待锁。

## 可重入锁、公平锁、乐观锁和悲观锁

- 可重入锁：已经获得锁后，再次尝试获取锁时不必重新申请锁。ReentrantLock和synchronized都是可重入锁。
- 公平锁：等待时间最久的线程会优先获得锁。非公平锁无法保证哪个线程取得锁。ReentrantLock默认为非公平锁，可以设置为公平锁，synchronized为非公平锁。
- 乐观锁：假设没有冲突，不加锁，更新时判断该数据是否过期，过期的话不进行数据更新，适用于读操作频繁的场景。乐观锁实现：AtomicInteger、AtomicLong、AtomicBoolean。
- 悲观锁：线程一旦获得锁，其他线程就挂起等待，适用于写操作频繁的场景。synchronized是悲观锁。

# 主要参考资料

https://zhuanlan.zhihu.com/p/148453122