---
title: 面试知识点一：Java基础
date: 2020-07-28 17:48:12
tags:
	- JAVA
categories:
	- JAVA
typora-root-url: ..
---

记录Java学习过程中的一些笔记。

<!--more-->

# Java基础

## 面向对象特性

- 封装：将对象的实现细节隐藏起来，通过公共的方法暴露对象的功能。
- 继承：继承使子类能够获取父类中的非private修饰的成员。
- 多态：继承，重写，向上转型。

## Java基本数据类型

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

## 接口与抽象类的区别

1. 抽象类里可以有非抽象方法，接口里不能有实现的方法（JDK 8开始可以有default 修饰的方法）。
2. 抽象类可以使用protected、public、default修饰符，接口里只能且默认都是是public。
3. 一个类只能继承一个抽象类，但是可以实现多个接口。
4. 抽象类里可以有类变量，接口里只能有静态常量static或static修饰的变量，且必须被初始化。

## ==与equals()区别

1. 对于基本类型，==是值比较；对于对象，==是内存地址比较。
2. equals()默认实现就是==，但是大多数类进行了重写，比如String、Interger等重写成了值比较。

## 为什么重写equals()时需要重写hashCode()

这个问题有个前提是该类需要被存储到HashSet、HashMap等散列表中，否则不需要用到hashCode()。

1. hashCode()默认返回值是根据对象的内存地址生成的一个整数，就算两个对象内容相同，也不会产生一样的hash值，因此需要重写。

2. 虽然不同的对象产生的hash值一定不同，但是具有相同hash值相同的对象却不一定相同，因此需要重写equals()对对象的内容进行进一步判断。

## 使用多线程有什么优势

1. 更有效地利用资源：当一个线程阻塞时，另一个线程可以执行其他事情。
2. 更快地响应：单线程只能串行地处理一个请求，然后监听下一个请求。多线程虽然处理速度不一定会更快，但是可以快速地进行线程切换，短时间内响应多个请求。

## static关键字

1. static修饰的变量为类变量，可以通过类名调用，静态变量存放在方法区（不能修饰方法中的局部变量）。

2. static修饰的方法为类方法，可以通过类名调用，静态方法不能被重写。

3. static修饰的代码块为静态代码块，会在类加载时被执行。

4. static修饰的内部类为静态内部类，不能使用外部类的非静态成员变量和方法。

## final关键字

1. final修饰类变量时，初始化后不能发生变化，必须在声明时初始化或在构造函数中初始化。
2. final修饰方法中局部变量时，只能赋值一次，赋值后不能发生变化。
3. final修饰的方法无法被重写（private方法也会被隐式地指定为final方法）。
4. final修饰的类无法被继承。

## 异常

![image-20200823112219462](../images/JAVA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%E2%80%94%E2%80%94JAVA%E5%9F%BA%E7%A1%80/image-20200823112219462.png)

- Error：程序无法处理的错误，通常是JVM出现的问题。
- Exception：程序本身能够处理的错误，

# Java集合

## 常用集合

- Collection
  - List：ArrayList、Vector、LinkedList
  - Set：HashSet、TreeSet、LinkedHashSet
- Map：HashMap、HashTable、LinkedHashMap、ConcurrentHashMap

## ArrayList与LinkedList的区别

1. 底层数据结构不同：ArrayList基于数组实现，LinkedList基于双向链表实现。
2. 随机访问和删除效率不同：ArrayList能快速随机访问，LinkedList只能从头遍历；ArrayList删除元素后需要移动后面的元素，LinkedList能快速删除元素。
3. 插入操作不同：ArrayList插入前，需要移动插入位置之后的元素；LinkedList插入时需要先遍历定位到元素。
4. 内存空间占用不同：ArrayList是一次性分配一批空间，而LinkedList是在需要时分配空间，但是LinkedList一个元素需要消耗更多的空间。

## ArrayList与Vector的区别

1. 线程安全性：ArrayList线程不安全，Vector中主要方法使用synchronized关键字保证线程同步。
2. 扩容因子不同，ArrayList在容量满时扩容为原来的1.5倍，Vector在容量满时扩容为原来的2倍。

## HashMap与HashTable的区别

1. 线程安全性：HashMap线程不安全，HashTable中主要方法使用synchronized关键字进行线程同步。
2. HashMap的key和value可以是null，HashTable的key和value不能为null。
3. 初始容量和扩容：HashMap默认初始容量为16，每次扩容为原来的2倍。HashTable默认初始容量为11，每次扩容为原来的2倍+1。
4. 底层数据结构：JDK 1.8以后的HashMap使用数组+链表/红黑树结构，而Hashtable使用数组+链表结构。

## ConcurrentHashMap与Hashtable的区别

ConcurrentHashMap是线程安全版HashMap，ConcurrentHashMap与HashTable的key和value不能为null。

1. 初始容量和扩容：HashMap默认初始容量为16，每次扩容为原来的2倍。HashTable默认初始容量为11，每次扩容为原来的2倍+1。
2. 底层数据结构：JDK 1.8以后ConcurrentHashMap采用数组+链表/红黑树结构，而Hashtable使用数组+链表结构。
3. 实现线程安全的方式：
   - ConcurrentHashMap：JDK1.8以前采用分段锁（Segment继承自ReentrantLock）；JDK 1.8及以后，锁的粒度更小，使用synchronized + CAS实现，数组被volatile关键字修饰，只对当前使用的链表头结点加锁。
   - Hashtable：使用synchronized整体加锁。

## Comparable与Comparator

实现java.lang.Comparable接口的类可以通过实现int compareTo(T o)进行两个类之间的比较。

实现java.util.Comparator接口的类为比较器类，需要实现int compare(T o1, T o2)，Arrays.sort()等方法可以通过传递一个比较器类实例来自定义比较规则。

# 多线程

## 什么是上下文切换

当前线程在执行完CPU时间片后切换到另一个线程前会先保存当前的运行状态，以便下次再切换回来。

## synchronized与ReentrantLock的区别

1. 底层实现不同
   - synchronized依赖于jvm实现，synchronized关键字在编译后会在代码块前后生成monitorenter和monitorexit指令，并通过计数器记录重入次数。
   - ReentrantLock依赖于API实现，基于AQS实现，AQS的state字段用于记录重入次数，通过CAS操作更新state的值。
2. ReentrantLock有一些特殊功能
   - 等待可中断：可以设置等待锁的超时时间。
   - 公平锁：按顺序获取锁。
   - 可绑定多个Condition：可以根据不同的Condition对线程进行分组，有选择性地进行线程通知；synchronized通常与wait()和notify()/notifyAll()结合实现线程间的通知。
3. synchronized在JDK 1.6及以后增加了许多优化，有偏向锁，轻量级锁和重量级锁三个级别。

## synchronized与volatile的区别

1. volatile只能修饰变量，而synchronized可以修饰方法和代码块。
2. 多线程情况下volatile不会发生阻塞，而synchronized可能会发生阻塞。
3. volatile能保证可见性，但不能保证原子性；synchronized两者都能保证。
4. volatile解决变量在多个线程直接的可见性，而synchronized解决多个线程之间访问资源的同步性。

## ThreadLocal内存泄漏问题

ThreadLocalMap中使用的key为ThreadLocal的弱引用，当ThreadLocal没有被ThreadLocalMap以外的对象引用时，ThreadLocal就在下一次GC时被回收，而ThreadLocalMap的value是强引用，可能出现key被回收但value还存在的情况，但是value已经不可访问，造成内存泄漏。因此使用玩ThreadLocal后应该手动调用remove()方法。

## 线程池执行execute()和submit()的区别

1. execute()用于提交不需要返回值的任务，参数为Runnable对象。

![image-20200824195854215](../images/Java%E9%9D%A2%E8%AF%95%E7%9F%A5%E8%AF%86%E4%B8%80%EF%BC%9AJava%E5%9F%BA%E7%A1%80/image-20200824195854215.png)

2. submit()用于提交需要返回值的任务，会返回一个Future对象，参数为Runnable对象，内部还是会调用execute()。

```java
public Future<?> submit(Runnable task) {
	if (task == null) throw new NullPointerException();
    RunnableFuture<void> ftask = new TaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

# JVM

## 内存区域（JDK 1.8及以后）

![image-20200824201627280](../images/Java%E9%9D%A2%E8%AF%95%E7%9F%A5%E8%AF%86%E4%B8%80%EF%BC%9AJava%E5%9F%BA%E7%A1%80/image-20200824201627280.png)

## 为什么要将永久代（PermGen）替换为元空间（MetaSpace）

永久代有一个JVM默认设置的固定大小，无法进行调整，而元空间使用的是直接内存，只受操作系统内存的限制。

## 对象创建过程

![image-20200824203844421](../images/Java%E9%9D%A2%E8%AF%95%E7%9F%A5%E8%AF%86%E4%B8%80%EF%BC%9AJava%E5%9F%BA%E7%A1%80/image-20200824203844421.png)

1. 类加载检查：首先根据new指令的参数在常量池中查找该对象的符号引用，如果不存在则先执行类加载过程：加载、验证、准备、解析、初始化。
2. 分配内存：根据对内存是否规整采用指针碰撞或空闲列表在堆中为新对象分配一块连续内存。
3. 初始化默认值：将内存空间中的实例字段初始化为默认值。
4. 设置对象头：在对象头中保存对象所属的类、哈希码、GC年龄等信息。
5. 执行\<init\>()：在以上工作完成后，在虚拟机中已经生成了一个完整的新对象。但通常还需要调用构造函数进行初始化。

## 如何判断一个类是无用的类

同时满足以下条件：

1. 该类的所有实例都已被回收。
2. 加载该类的ClassLoader已被回收。
3. 该类对应的java.lang.Class对象没有被引用。

