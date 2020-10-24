---
title: Java并发编程
date: 2020-08-15 20:15:00
tags: 
	- 并发
	- JAVA
categories:
	- Java并发编程
typora-root-url: ..
---

Java并发编程。

<!--more-->

# 线程基础

## 线程创建的方式

- Thread：继承**Thread**类并重写run方法，直接使用this可以获取当前线程。
- Runnable：实现**Runnable**接口并传递给**Thread**。可以继承其他类，多个线程可以共用一个代码逻辑。
- FutureTask：实现**Callable**接口传递给**FutureTask**，然后把FutureTask传递给**Thread**。可获取返回结果。

## 线程通知与等待

**Object**类中的通知与等待系列函数，这些方法**获得该对象的锁后才能调用**：

- wait()：线程阻塞并释放当前对象的锁。返回情况：
  - 其他线程调用**锁对象**的notify()或notifyAll()。
  - 其他线程调用**该线程**的interrupt()，该线程抛出InterruptedException异常返回。
- notify()：随机唤醒一个被挂起的线程来竞争锁。
- notifyAll()：唤醒所有被挂起的线程来竞争锁。

**Thread**类中的通知与等待系列函数：

- join()：阻塞当前线程，等待目标线程执行完毕。
- sleep()：当前线程让出CPU使用权，被阻塞挂起，但是不会释放锁。
- yield()：当前线程让出CPU使用权，不会被阻塞，处于就绪状态。

## 守护线程

JVM会等待所有用户线程结束后才退出，而守护线程（Daemon Thread）是否结束不影响JVM的退出。

## 线程本地变量

### ThreadLocal

访问ThreadLocal变量的线程会在线程中有一个本地副本，操作这个变量不需要与主内存同步。

实现原理：每个**Thread实例**中有一个ThreadLocalMap类型的threadLocals变量，threadLocals以当前ThreadLocal的实例对象引用（this）为key对数据进行存储与读取。

```java
ThreadLocal<String> var = new ThreadLocal<>();
var.set("test");
System.out.println(var.get());
```

内存泄漏问题：ThreadLocalMap的key是对ThreadLocal对象的弱引用，当其他地方没有对ThreadLocal对象的引用时，ThreadLocal对象会被回收，但是对应的value没有被回收，这时候就存在key为null但是value不为null的项。解决方法是使用完毕后调用remove()。

### InheritableThreadLocal

InheritableThreadLocal继承自ThreadLocal，可以让子线程访问父线程中设置的本地变量。

实现原理：每个Thread实例中保存了类似threadLocals的inheritableThreadLocals变量，在当前线程创建新线程时，把当前线程的inheritableThreadLocals变量赋给新线程。

```java
Thread parent = currentThread(); // parent为当前线程
...
if (parent.inheritableThreadLocals != null) {
	this.inheritableThreadLocals = // this为当前线程正在初始化的新线程
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
}
```

### ThreadLocalRandom

每个Thread实例维护一个线程级别的种子变量threadLocalRandomSeed，避免了竞争。

## 内存语义

多线程情况下，可能会出现**缓存一致性**问题（其他线程修改了变量的值并刷新到了主内存中，但是当前线程的工作内存没有及时刷新，与主内存的值不一致）。

锁的可见性就是通过缓存无效化解决这一问题。

### synchronized

- 加锁时，先清除工作内存变量值，然后从主内存读取。

- 释放锁时，把代码块里的共享变量修改刷新到主内存。

### volatile

内存语义

- 读volatile变量时，先清除工作内存变量值，然后从主内存读取。

- 写volatile变量时，把共享变量刷新到主内存。

禁止重排序

- 读volatile变量时，确保读之后的操作不会被重排序到读之前。
- 写volatile变量时，确保写之前的操作不会被重排序到写之后。

## 锁的类别

- 悲观锁：认为共享变量很容易发生竞争，因此对共享变量进行操作时会加锁。
- 乐观锁：任务共享变量一般情况下不会发生竞争，所以在操作前不会加锁，而是在更新时才进行冲突检测。
- 公平锁：线程获取锁的顺序是按照现场请求锁的时间决定的，会带来性能开销。
- 非公平锁：线程获取锁的顺序不确定。
- 独占锁：保证任何时候只能有一个线程获得锁。是一种悲观锁。
- 共享锁：允许多个线程同时进行读操作。是一种乐观锁。
- 可重入锁：线程再次获取当前已有的锁不会被阻塞。通过在锁内部维护一个线程标志和计数器实现。
- 自旋锁：执行忙循环检测是否可以获得锁。

# JUC并发包（java.util.concurrent）

## 原子操作类

### AtomicInteger、AtomicLong、AtomicBoolean

属于java.util.concurrent.atomic包

- 基于Unsafe的**CAS**保证数据操作的**原子性**。
- 通过对value加**volatile关键字**保证**可见性**和**有序性**。

### LongAdder、DoubleAdder

AtomicLong等类在有大量线程竞争时，会不断自旋尝试CAS操作，造成了CPU资源浪费，LongAdder类在内部维护多个Cell变量（一个Cell数组），每个Cell有一个**volatile修饰**的long型变量。当线程在当前Cell变量CAS失败后，可以在其他Cell上进行CAS尝试，最后把**所有Cell的值累加后再加上base**返回。

使用`@sun.misc.Contended`注解修饰Cell类**避免伪共享**。

调用sum()返回当前的值，但是没有加锁，所以返回的值不一定准确。

## 并发List

### CopyOnWriteArrayList

是一个线程安全的ArrayList，使用了写时复制策略，对其进行的修改操作都是在一个复制的数组上进行的。

内部使用一个**volatile修饰**的数组来存放具体元素。

get()不加锁。add()、set()、remove()时加**ReentrantLock**锁，对数组的一个快照进行操作，不影响其他线程读取原数组，然后更新原数组的引用。

迭代器的弱一致性：返回迭代器后，其他线程对list的增删改对迭代器是不可见的。

CopyOnWriteArraySet底层是使用CopyOnWriteArrayList实现的。

## 锁原理

### LockSupport工具类

主要作用是挂起和唤醒线程，是创建锁和其他同步类的基础。基于Unsafe类的park()和unpark()实现。

LockSupport类与每个使用它的线程会关联一个许可证，默认情况下调用LockSupport类方法的线程是不持有许可证的。

- park()：调用线程如果已经获得许可证，则调用时会马上返回，否则会被阻塞挂起。
- unpark(Thread thread)：如果线程没有持有许可证，则让线程持有，如果线程因调用park()挂起，则唤醒线程。

### 抽象同步队列AQS（AbstractQueueSynchronizer）

![image-20200903224211947](/images/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/image-20200903224211947.png)

是实现同步器的基础组件，JUC中锁的底层就是使用AQS实现的。

AQS是一个双向队列，head和tail记录队首和队尾结点。

#### 静态内部类Node

- Thread thread：**volatile**修饰，存放进入AQS的线程。
- Node SHARED：标记该线程是获取**共享资源**时被阻塞后放入AQS的。
- Node EXCLUSIVE：标记该线程是获取**独占资源**时被阻塞后加入AQS的。
- Node pre, next：**volatile**修饰，记录前驱和后继结点。

#### AQS维护了一个volatile修饰的状态信息state

- 在ReentrantLock中：state表示当前线程获取锁的可重入次数。
- 在ReentrantReadWriteLock中：state高16位表示读锁的次数，低16位表示获取到写锁的线程可重入次数。
- 在Semaphore中：state用于表示当前可用信号的个数。
- 在CountDownLatch中：state表示计数器的值。

#### 内部类ConditionObject

![image-20200819100243950](/images/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/image-20200819100243950.png)

实现了Condition接口，每个条件变量对应一个条件队列，用于存放条件变量的await()后被阻塞的线程。

Node firstWaiter, lastWaiter分别记录了条件队列的头、尾结点。

**Condition**类中的通知与等待系列函数，这些方法**在调用锁的lock()方法获取锁后才能调用**：

- await()：将当前线程封装成一个Node结点插入到条件队列的尾部，释放当前线程获取的锁，并调用LockSupport.park(this)挂起当前线程。
- signal()：把队头线程结点从条件队列中移除并放入AQS的阻塞队列里面，然后激活这个线程。
- signalAll()：移除条件队列中所有节点并放入AQS的阻塞队列里面。

#### 其他锁在基于AQS实现时，关键是定义state的操作方法

- 独占方式：acquire()，acquireInterruptibly()，release()。

  获取与释放资源：

  - 线程调用acquire()获取资源：方法内部调用**tryAcquire()**尝试设置state的值，成功则直接返回，失败则将当前线程封装为一个Node结点插入到AQS队列的尾部，并调用LockSupport.park(this)挂起自己。

  - 线程调用release()释放资源：方法内部调用**tryRelease()**尝试设置state的值，然后调用LockSupport.unpark(thread)激活AQS队列中的一个线程，被激活的线程调用**tryAcquire()**尝试获取锁。

  acquireInterruptibly()：忽略中断，不对中断进行响应。

- tryAcquire()和tryRelease()需要由具体的子类根据需求来实现。

- 共享方式：acquireShared()，acquireSharedInterruptibly()，releaseShared()。获取、释放资源流程与独占方式类似。

### ReentrantLock

![image-20200903223938171](/images/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/image-20200903223938171.png)

基于AQS实现，state表示该线程获取该锁的可重入次数，可以设置为公平锁或非公平锁。

- 非公平锁：当一个线程第一次获取该锁时会尝试使用**CAS**设置state的值为1，如果成功则设置锁持有者为当前线程，否则检查锁的持有线程是不是当前线程，是的话更新state加一，否则将当前线程放入AQS队列。
- 公平锁：线程获取锁时，首先检查锁是否已被占有：如果没有被占有，再检查AQS队列中是否有其他线程在等待，如果没有则尝试获取锁，如果有则将当前线程加入到AQS队列尾部。如果已被占用，则检查持有锁的线程是不是当前线程，是的话则更新state加一，否则将当前线程加入到AQS队列尾部。

释放锁时将state值减一，若state为0则表示释放锁。

### ReentrantReadWriteLock

![image-20200903224043298](/images/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/image-20200903224043298.png)

读写分离策略，内部维护了两个锁：ReadLock和WriteLock。

使用AQS实现，state的高16位表示获取到读锁的次数，低16位表示获取到写锁的线程可重入次数。

- 写锁WriteLock：独占可重入锁，如果当前已有其他线程持有读锁或写锁，则当前线程会被阻塞挂起。
- 读锁ReadLock：如果当前没有其他线程持有写锁，则可以获取读锁，state的高16加1。如果有其他线程持有写锁，则当前线程会被阻塞。

### StampedLock

提供了三种模式的读写控制，调用获取锁的函数时，会返回一个long型stamp表示锁的状态，当调用释放锁和转换锁的方法时需要传入这个stamp值。

StampedLock提供三种读写模式的锁：

- 写锁writeLock：独占锁，不可重入。当目前没有其他线程持有读锁或者写锁时才能获取到该锁。
- 悲观读锁readLock：共享锁，不可重入。其他线程可以加读锁，不能加写锁。
- 乐观读锁tryOptimisticRead：没有显式的加锁释放锁，在具体操作数据前根据stamp验证期间是否有其他线程持有了写锁。

## 非阻塞队列

### ConcurrentLinkedQueue

继承自AbstractQueue，使用单向链表实现，无界队列，入队和出队操作使用**CAS**来实现线程安全。

- 队列中保存了**volatile**关键字修饰的head和tail结点。

- 每个Node包括**volatile**修饰的元素和next结点。

添加元素时通过CAS操作来保证只有一个线程可以成功追加元素到队列尾部，失败的线程会循环尝试添加元素。

通过无限循环的CAS尝试来代替阻塞，**使用CPU资源换阻塞带来的用户态与内核态切换、线程切换等开销**。

## 阻塞队列

![image-20200901234217695](/images/Java%E5%B9%B6%E5%8F%91/image-20200901234217695.png)

都继承自AbstractQueue类，除LinkedTransferQueue外都实现了**BlockingQueue**接口。

### LinkedBlockingQueue

![image-20200904192818763](/images/Java%E5%B9%B6%E5%8F%91/image-20200904192818763.png)

使用单向链表实现，无界队列。

- 两个ReentrantLock实例保证操作的原子性：
  - putLock：调用put()、offer()等操作时在内部会尝试获取该锁，保证同时**只有一个线程可以操作尾结点**。
  - takeLock：调用take()、poll()等操作时在内部会尝试获取该锁，保证同时**只有一个线程可以操作头结点**。
- notEmpty为takeLock的条件变量，notFull为putLock的条件变量，是生产者-消费者模型。

### ArrayBlockingQueue

![image-20200904192834599](/images/Java%E5%B9%B6%E5%8F%91/image-20200904192834599.png)

使用数组实现，有界队列，构造时必须传入队列大小。与LinkedBlockingQueue不同的是，ArrayBlockingQueue采用一个全局锁，加锁粒度更大。

- 数组items用于存放队列元素。
- 一个**ReentrantLock**实例lock用于保证同时只有一个线程可以进行出队入队操作。
- 两个lock的条件变量：
  - notEmpty：队列空时，出队线程被阻塞挂起放入该条件队列。
  - notFull：队列满时，入队线程被阻塞挂起放入该条件队列。

### PriorityBlockingQueue

![image-20200904200910774](/images/Java%E5%B9%B6%E5%8F%91/image-20200904200910774.png)

使用平衡二叉树堆（数组实现）实现，带优先级的无界队列。

- 数组queue用于存放队列元素。
- 一个**volatile**修饰的int类型自旋锁allocationSpinLock保证同时只有一个线程可以扩容队列：
  - 0表示当前没有进行扩容。
  - 1表示当前正在扩容。
- 一个全局锁**ReentrantLock**实例lock用于保障同时只有一个线程可以进行出队入队操作。
- 一个lock的条件变量notEmpty用于实现出队的阻塞模式，由于是无界队列，入队时是非阻塞的。

使用数组存放队列元素，一个自旋锁通过CAS操作来保证同时只有一个线程可以扩容队列，一个ReentrantLock控制只有一个线程可以进行入队、出队操作。notEmpty条件变量控制出队同步。

## 线程池ThreadPoolExecutor

在执行大量异步任务时线程池能够提供较好的性能，线程池里面的线程是可复用的，不需要每次执行异步任务时都重新创建和销毁线程。

- 参数ctl是一个AtomicInteger变量，高3位表示线程池状态，低29位记录线程个数。
- 内部类Worker继承AQS和Runnable接口，一个Worker封装了一个线程，并实现了简单不可重入独占锁，state为0时表示锁未被获取，state为1时表示锁已被获取。
- HashSet\<Worker\>类型的workers用于存储线程。

使用完线程池需要调用shutdown()关闭线程池，否则会导致主线程已经退出，但是JVM仍然存在。

### 线程池状态

- **RUNNING**：接受新任务并处理阻塞队列里的任务。
- **SHUTDOWN**：拒绝新任务但是**处理阻塞队列里的任务**。
- **STOP**：拒绝新任务并**抛弃阻塞队列里的任务，同时中断正在处理的任务**。
- **TIDYING**：所有任务执行完毕后线程数为0，即将调用terminated方法。
- **TERMINATED**：终止状态。

### 线程池构造参数

- corePoolSize：线程池核心线程个数。
- maximumPoolSize：线程池最大线程数量。
- workQueue：保存等待执行的任务的阻塞队列：
  - 基于数组的有界阻塞队列ArrayBlockingQueue。
  - 基于链表的无界阻塞队列LinkedBlockingQueue。
  - 最多只有一个元素的阻塞队列SynchronousQueue。
  - 优先级阻塞队列PriorityBlockingQueue。
- ThreadFactory：创建线程的工厂。
- RejectedExecutionHandler：队列满并且线程个数达到最大后采取的策略。
  - AbortPolicy：抛出异常。
  - CallerRunsPolicy：使用调用者所在的线程来运行任务。
  - DiscardOldestPolicy：调用poll()丢弃一个任务，执行当前任务。
  - DiscardPolicy：丢弃任务，不抛出异常。
- keepAliveTime：线程数量比核心线程数量多且闲置时，表示闲置线程的最大存活时间。
- TimeUnit：存活时间的单位。

### 创建线程池

- **newFixedThreadPool**：核心和最大线程数都为参数nThread，阻塞队列长度为整型最大值，可以设置存活时间和阻塞队列类型。
- **newSingleThreadExecutor**：**核心和最大线程数都为1**，阻塞队列长度为整型最大值，可以设置存活时间和阻塞队列类型。
- **newCachedThreadPool**：按任务创建线程，初始线程个数为0，最多为整型最大值，阻塞队列为同步队列且加入同步队列的任务会被马上执行，**同步队列中对多只有一个任务**。

### 方法

- execute(Runnable command)：提交任务command到线程池执行。如果当前线程池的线程数量小于核心线程数量，则通过CAS操作向workers里面新增一个核心线程执行该任务。

  ![image-20200904225713537](/images/Java%E5%B9%B6%E5%8F%91/image-20200904225713537.png)

## ScheduledThreadPoolExecutor

可以在指定一定延迟时间后或者定时进行任务调度执行的线程池。

使用DelayQueue来存放任务，任务分为三种：

- 一次性执行任务：执行完毕就结束了。
- fixed-delay任务：同一任务在多次执行之间间隔固定时间。
- fixed-rate任务：保证按照固定的频率执行。

## 线程同步器

### CountDownLatch

![image-20200904230700998](/images/Java%E5%B9%B6%E5%8F%91/image-20200904230700998.png)

使用AQS实现，把计数器的值赋给AQS的状态变量state，多个线程调用countDown()实际是原子性递减state。当线程调用await()方法后会被放入AQS的阻塞队列等待计数器为0再返回。

以线程数量为参数创建一个CountDownLatch实例，主线程调用countDownLatch.await()后会被阻塞，子线程执行完毕后调用countDownLatch.countDown()使countDownLatch内部的计数器减1，计数器为0时主线程的await()方法返回，表示所有子线程已执行完毕。

使用线程池管理线程时一般是直接添加Runnable到线程池，这时候就无法调用线程的join()，而CountDownLatch使我们对线程同步有更灵活的控制。

### 回环屏障CyclicBarrier

![image-20200904231215070](/images/Java%E5%B9%B6%E5%8F%91/image-20200904231215070.png)

CyclicBarrier可以让一组线程全部达到一个状态后再全部同时执行。

线程调用await()后会被阻塞，这个阻塞点称为屏障点，等所有线程都调用了await()后，线程们就会冲破屏障，继续向下执行。

基于ReentrantLock实现，本质底层还是基于AQS，parties记录线程数量，count记录还有多少个线程没有到达屏障点。使用ReentrantLock保证count更新的原子性。

### 信号量Semaphore

![image-20200904231439695](/images/Java%E5%B9%B6%E5%8F%91/image-20200904231439695.png)

也是一个同步器，计数器是递增的并且可以指定一个初始值，在需要同步的地方调用acquire()指定需要达到的计时器值，在线程内部调用release()使计数器加1。

也是使用AQS实现的，获取信号量时有公平和非公平策略。

# NIO模型——Tomcat的NioEndPoint

![image-20200904233842705](/images/Java%E5%B9%B6%E5%8F%91/image-20200904233842705.png)

使用队列把接受请求与处理请求操作进行解耦，实现异步处理。实际Poller中维护一个ConcurrentLinkedQueue用于缓存任务，其本身也是一个多生产者-单消费者模型。

- Acceptor：套接字接受线程，用于接受用户的请求，并把请求封装为事件放入Poller的队列。只有一个。
- Poller：套接字处理线程，每个Poller内部都有一个独有的队列，Poller线程从自己的队列中获取事件任务交给Worker进行处理。Poller线程的个数与处理器的核数有关。
- Worker：实际处理请求的线程。

## 生产者——Acceptor线程

Acceptor线程中一直循环接收客户端连接请求，当成功获取到一个连接套接字后，将其封装为一个Channel对象并注册为事件存放到Poller对象的队列中。

## 消费者——Poller线程

Poller线程中从事件队列中获取一个事件，遍历所有注册的Channel并对感兴趣的事件进行处理。