---
title: JAVA后台开发项目学习
date: 2020-08-03 20:15:00
tags: 
	- JAVA
	- 后台
categories:
	- JAVA 
typora-root-url: ../..
---

![image-20200809192659015](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200809192659015.png)

<!--more-->

dao=data access object

# 数据库

## user表

```mysql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `password` varchar(50) DEFAULT NULL,
  `salt` varchar(50) DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  `type` int DEFAULT NULL COMMENT '0-普通用户; 1-超级管理员; 2-版主;',
  `status` int DEFAULT NULL COMMENT '0-未激活; 1-已激活;',
  `activation_code` varchar(100) DEFAULT NULL,
  `header_url` varchar(200) DEFAULT NULL,
  `create_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_username` (`username`(20)),
  KEY `index_email` (`email`(20))
) ENGINE=InnoDB AUTO_INCREMENT=158 DEFAULT CHARSET=utf8
```

## login_ticket表

```mysql
CREATE TABLE `login_ticket` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` int NOT NULL,
  `ticket` varchar(45) NOT NULL,
  `status` int DEFAULT '0' COMMENT '0-有效; 1-无效;',
  `expired` timestamp NOT NULL,
  PRIMARY KEY (`id`),
  KEY `index_ticket` (`ticket`(20))
) ENGINE=InnoDB AUTO_INCREMENT=24 DEFAULT CHARSET=utf8
```

## discuss_post表

```mysql
CREATE TABLE `discuss_post` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` varchar(45) DEFAULT NULL,
  `title` varchar(100) DEFAULT NULL,
  `content` text,
  `type` int DEFAULT NULL COMMENT '0-普通; 1-置顶;',
  `status` int DEFAULT NULL COMMENT '0-正常; 1-精华; 2-拉黑;',
  `create_time` timestamp NULL DEFAULT NULL,
  `comment_count` int DEFAULT NULL,
  `score` double DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=289 DEFAULT CHARSET=utf8
```

## comment表

评论的评论称为回复。

查询某个帖子的评论时，给定type值和帖子id即可。

查询某个评论的回复时，给定type值和评论id即可。同时若targetId不为0，则还需要查询回复的目标用户。

```mysql
CREATE TABLE `comment` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_id` int DEFAULT NULL,
  `entity_type` int DEFAULT NULL, // 评论目标的类型（帖子或评论）
  `entity_id` int DEFAULT NULL, // 评论目标的ID
  `target_id` int DEFAULT NULL, // 回复的目标用户ID
  `content` text,
  `status` int DEFAULT NULL,
  `create_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_user_id` (`user_id`) /*!80000 INVISIBLE */,
  KEY `index_entity_id` (`entity_id`)
) ENGINE=InnoDB AUTO_INCREMENT=249 DEFAULT CHARSET=utf8
```

## message表

```mysql
CREATE TABLE `message` (
  `id` int NOT NULL AUTO_INCREMENT,
  `from_id` int DEFAULT NULL, // 系统通知为1
  `to_id` int DEFAULT NULL,
  `conversation_id` varchar(45) NOT NULL, // from_id和to_id的组合，小的id在前。
  `content` text,
  `status` int DEFAULT NULL COMMENT '0-未读;1-已读;2-删除;',
  `create_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_from_id` (`from_id`),
  KEY `index_to_id` (`to_id`),
  KEY `index_conversation_id` (`conversation_id`)
) ENGINE=InnoDB AUTO_INCREMENT=368 DEFAULT CHARSET=utf8
```

## Redis表

### 某个实体获得的赞

- **key：**"like:entity" + entityType + entityId

- **value：**set存放点赞的userId

### 某个用户获得的赞（个人主页显示）

- **key：**"like:user" + userId
- **value：**string存放获得赞的数量

### 某个用户关注的用户

- **key：**"followee:" + userId + ":user"
- **value:** zset存放关注的用户ID

某个用户拥有的粉丝

- **key：**"follower:user" + userId 
- **value：**zset存放拥有的粉丝ID

# Spring

@Bean：修饰方法，该方法返回的对象会被装配到Bean容器中。

## Bean的管理

- @PostConstruct：修饰方法，在Bean的构造函数之后调用。
- @PreDestroy：修饰方法，在Bean销毁之前调用。

# HTTP请求

第14节 SpringMVC入门

## Get传递参数的方式

- 在Url中通过?后接参数信息，如/student?id=1。这种方式可以通过`@RequestParam(name = "id", required = false, defaultValue = "1")`在服务器中对应的方法参数中获取。
- 在url中传递参数，如/student/1，同时`@RequestMapping(path = "/student/{id}")`。这种方式可以通过`@PathVariable("id")`在服务器中对应的方法参数中获取。

## Post传递参数的方式

前端表单中的参数名与后台对应方法的参数名一致即可。

## 转发与重定向

重定向：返回给浏览器**302状态码**和一个url，浏览器再次访问该url。如注册成功后自动跳转到登陆页面。

# 日志

SpringBoot默认启用的日志工具为LOGBack（org.slf4j.Logger）

日志级别：trace，debug，info，warn，error

# 邮件

1. 导入Spring Boot Mail Starter。
2. 邮箱配置（域名，端口，发件人账号密码）。
3. 构建MimeMessage（发件人，收件人，主题，内容）。
4. 注入JavaMailSender实例，并调用其send()方法发送邮件。

# 工具包

- Apache Commons Lang：字符串工具包。
- Kaptcha：生成验证码图片。
- Fastjson：生成或处理json数据。

# Cookie和Session

## Cookie

数据存在客户端。

```java
@RequestMapping(path = "/cookie/set", method = RequestMethod.GET)
@ResponseBody
// 生成一个键为key，值为value的cookie，并添加到response中返回给浏览器
public String setCookie(HttpServletResponse response) {
	Cookie cookie = new Cookie("key", "value");
	// 设置cookie生效的路径范围
	cookie.setPath("/community/alpha");
	// 设置cookie的生存时间（秒）
	cookie.setMaxAge(60 * 10);
	// 发送cookie
	response.addCooikie(cookie);
	return "set cookie";
}

@RequestMapping(path = "/cookie/get", method = RequestMethod.GET)
@ResponseBody
// 获取键为key的cookie的value，并传递给参数value
public String getCookie(@CookieValue("key") String value) {
    System.out.println(value);
    return "get cookie";
}
```

## Session

数据存在服务器内存，只传递一个SessionID。

```java
@RequestMapping(path = "/session/set", method = RequestMethod.GET)
@ResponseBody
// 自动注入一个HttpSession实例，然后设置信息，（JSessionID）传给浏览器
public String setSession(HttpSession session) {
    session.setAttribute("id", 1);
    session.setAttribute("name", djs);
    return "set session";
}

@RequestMapping(path = "/session/get", method = RequestMethod.GET)
@ResponseBody
public String getSession(HttpSession session) {
    System.out.println(session.getAttribute("id"));
    System.out.println(session.getAttribute("name"));
    return "set session";
}
```

分布式Session：

- 粘性Session：同一个IP分配到同一个服务器。
- 同步Session：每个服务器的Session通过同步保持一致。
- 共享Session：单独一台服务器存放Session数据，其他服务器向该服务器请求Session。
- 基于数据库：使用MySQL或者Redis集群存储Session，其他服务器向该集群请求Session。

# 拦截器

对于一些需要登录才能访问或者需要一定权限的路径如个人信息，需要使用拦截器检查登录状态。

在用户浏览网页时，需要检查用户登陆状态，通过拦截器实现。

1. 实现HandlerInterceptor接口中的三个方法：

   - preHandler()：在Controller之前执行。获取Cookie中的凭证id，然后在数据库中查询凭证有效状态并根据凭证获取到User对象并保存到ThreadLocal变量hostHolder中。

   - postHandler()：在Controller之后，模板引擎之前执行。从hostHolder获取到User并传给模板引擎。

   - afterCompletion()：在模板引擎之后执行。清除hostHolder。
2. 在实现WebMvcConfigurer接口的类中addInterceptors()配置拦截器：拦截哪些路径，不拦截哪些路径。

# 检查登录状态

通过自定义注解修饰需要拦截的方法。

```java
// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginRequired {
    
}
```

在拦截器中检查拦截到的方法是否包含自定义注解，然后进行登录状态检查。

```
LoginRequired loginRequired = method.getAnnotation(LoginRequired.class);
```

# 敏感词过滤

## 前缀树

查找效率高，消耗内存大

## 敏感词过滤器

- 定义前缀树
- 根据敏感词，初始化前缀树
- 编写过滤敏感词的方法

# Spring事务管理

- 声明式事务
  - 通过XML配置，声明某方法的事务特征。
  - 通过注解，声明某方法的事务特征。

```java
@Transactional(isolation = Isolation.READ_COMMMITED, propagation = Propagation.REQUIRED)
public void method() {
    // 实现事务
}
```

- 编程式事务：通过TransactionTemplate管理事务，并通过它执行数据库操作。

```java
@Autowired
private TransactionTemplate template;
public void method() {
    template.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITED);
    template.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    return template.execute(new TransactionCallback<Object>() {
        @Override
        public Object doInTransaction(TransactionStatus status) {
            // 实现事务
            return null;
        }
    });
}
```

隔离级别，传递机制。

# 显示评论

分页显示，评论，回复。

# 添加评论

**事务操作**：增加评论并更新帖子的评论数量。

# 私信列表

显示未读消息数量，显示最新一条私信。

# 发送私信

异步发送

# 统一处理异常

**@ControllerAdvice**

- 用于修饰类，表示该类是Controller的全局配置类。

- 在此类中，可以对Controller进行全局配置：异常处理方案、绑定数据方案、绑定参数方案。

  - **@ExceptionHandler**：用于修饰方法，该方法会在Controller出现异常后调用，用于处理捕获到的异常。

  - **@ModelAttribute**：用于修饰方法，该方法会在Controller方法执行前被调用，用于为Model对象绑定参数。

  - **@DataBinder**：用于修饰方法，该方法会在Controller方法执行前被调用，用于绑定参数的转换器。

# 统一记录日志

AOP（Aspect Oriented Programing，面向方面（切面）编程），是对OOP的补充，可以进一步提高编程的效率。

![image-20200804231100773](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200804231100773.png)

应用场景：多个模块共享权限检查，记录日志，事务管理等操作。

## AOP的实现

AspectJ：一种新语言，扩展了Java，定义了AOP语法，在编译期织入代码。它有一个专门的编译器，用来生成遵守Java字节码规范的class文件。

Spring AOP：使用纯Java实现，在运行时通过代理方式织入代码，**只支持方法类型**的连接点，支持AspectJ的集成。

## Spring AOP

**JDK动态代理：**Java提供的动态代理技术，可以在运行时创建接口的代理实例。Spring AOP默认采用这种方式，在接口的代理实例中织入代码。（代理模式）

**CGLib动态代理：**采用底层的字节码技术，在运行时创建子类代理实例。当目标对象不存在接口时，Spring AOP会采用这种方式，在子类实例中织入代码。

```java
// 修饰一个类
@Aspect
// 修饰一个方法，括号内为切入点模式
@Pointcut("execution (* com.djs.community.service.*.*(...))")
// 修饰一个方法，在切入点之前执行。
@Before("pointcut()")
// 修饰一个方法，在切入点之后执行。
@After("pointcut()")
// 修饰一个方法，在切入点返回之后执行。
@AfterReturning("pointcut()")
// 修饰一个方法，在抛出异常时执行。
@AfterThrowing("pointcut()")
// 修饰一个方法，在目标方法前后执行。
@Around("pointcut()")
```

# Redis

Redis是一款基于键值对的NoSQL数据库，它的key是string，value支持多种数据结构：字符串（Strings），哈希（Hashes），列表（Lists），集合（Sets），有序集合（Sorted Sets）等。

Redis将所有数据都存放在内存中，读写速度快。同时，Redis还可以将内存中的数据以**快照**或者**日志**的形式保存在硬盘上，保证数据的安全。

应用场景：缓存、排行榜、计数器、社交网络、消息队列等。

`keys *`：查看所有的key。

`keys test*`：查看test开头的key。

`type key`：查看某个key的类型。

`exists key`：查看key是否存在。

`expire key seconds`：设置key在seconds秒后自动删除。

## String

存：`set key value [EX seconds] [PX milliseconds] [NX|XX] `

取：`get key`

## Hash

存：`hset key field value`

取：`hget key field`

## List

支持队列，栈模拟

从左存：`lpush key value [value ...]`

从左按索引查看：`lindex key index`

从左按索引范围查看：`lrange key start stop`

从左弹出一个值：`lpop key`

长度：`llen key`

## Set

存：`sadd key member [member ...]`

统计元素个数：`scard key`

随机弹出一个值：`spop key [count]`

查看集合中元素：`smembers key`

## Sorted Sets

按`score`排序

存：`zadd key [NX|XX] [CH] [INCR] score member [score member ...]`

统计元素个数：`zcard key`

查询某个元素的score：`zscore key member`

查询某个元素的排序：`zrank key member`

取排序范围的值：`zrange key start stop [WITHSCORES]`

# 点赞

异步点赞，点赞信息存在Redis。

- **key：**"like:entity" + entityType + entityId

- **value：**set存放点赞的userId

# 收到的赞

事务管理：记录帖子的点赞数量同时记录该贴子的作者收到的赞。

- **key：**"like:user" + userId

- **value：**string存放获得赞的数量

# 关注、取消关注

关注信息存在Redis。

异步（取消）关注，**事务操作**：更新user的关注信息，更新目标的粉丝信息。

# 关注列表、粉丝列表

Redis查询。

# 使用Redis优化登录模块

- 使用Redis存储验证码（kaptcha）：需要频繁刷新和访问，只需要保存一段时间，避免分布式部署时的Session共享问题。
- 使用Redis存储登录凭证：每次处理请求时都需要查询用户的登录凭证，访问频率非常高。
- 使用Redis缓存用户信息：每次请求时都需要根据凭证查询用户。

# Kafka构建TB级异步消息系统

## 阻塞队列

BlockingQueue接口：解决线程通信问题。

生产者消费者模式：生产者——产生数据的线程，消费者——使用数据的线程。

实现类：ArrayBlockigQueue、LinkedBlockingQueue、PriorityBlockingQueue、SynchronousQueue、DelayQueue等。

## Kafka

Kafka是一个分布式的流媒体平台，应用：消息系统、日志收集、用户行为追踪、流式处理等。

特点：高吞吐量、消息持久性（顺序存在硬盘，硬盘的顺序读写速度高于内存的随机读写）、高可靠性（分布式）、高扩展性。

术语：

- Broker：Kafka的每台服务器叫Broker
- Zookeeper：管理服务器集群
- Topic：采用发布订阅模式，每条消息都要发送到指定的Topic上
- Partition：Topic多个分区，提高并发执行能力
- Offset：消息在Partition中的索引
- Leader Replica：主副本可以处理请求
- Follow Replica：随从副本是主副本的备份。

启动Zookeeper：

1. 切换到Kafka安装目录`D:\kafka_2.12-2.5.0`
2. 输入命令`bin\windows\zookeeper-server-start.bat config\zookeeper.properties`启动Zookeeper。

启动Kafka：

1. 切换到Kafka安装目录`D:\kafka_2.12-2.5.0`
2. 输入命令`bin\windows\kafka-server-start.bat config\server.properties`启动kafka。

创建Topic：

1. 切换到目录`D:\kafka_2.12-2.5.0\bin\windows`。
2. 输入命令`kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test1`创建一个名为test的topic。

## Spring整合Kafka

引入依赖spring-kafka并配置kafka。

生产者：`kafkaTemplate.send(topic, data);`

消费者：注解`@KafkaListener(topic = {"test"})`用于修饰监听名为test的topic里面的消息。

# 发布系统通知

评论、点赞、关注后，通过Kafka异步地在对应的topic发布通知。

# Elasticsearch（ES）

ES是一个分布式的、Restful风格的搜索引擎，支持对**各种类型**的数据检索，可以提供**实时**的搜索服务，便于水平扩展，每秒可处理PB级别的海量数据。

术语：

- 索引、类型、文档、字段（对应MySQL中的数据库，表，行，列）
- 集群、结点、分片、副本

安装配置：

- 安装ES：支持分词搜索

- 安装分词插件`elasticsearch-analysis-ik-6.4.3`
- 安装Postman（非必须）：便于向ES服务器发送HTTP请求

## Spring整合ES

引入依赖spring-boot-starter-data-elasticsearch

## 启动ES服务器

运行文件`D:\elasticsearch-6.4.3\bin\elasticsearch.bat`文件

# 开发论坛搜索功能

将帖子保存至ES服务器，从ES服务器搜索帖子。

发布帖子，增加评论时，发布事件，异步地将帖子提交到ES服务器（结合kafka）

# 提高安全性——Spring Security

Spring Security是一个专注于为Java应用程序提供身份认证和授权的框架，强大之处在于轻松扩展以满足自定义需求。

特点：对身份认证和授权提供全面的、可扩展的支持；防止各种攻击如会话固定攻击、点击劫持、CSRF攻击等；支持与Servlet API、Spring MVC等Web技术集成。

DispatcherServlet是Spring MVC的核心组件，用于给Controller分发请求，分发过程中可以通过拦截器进行拦截。

## 转发与重定向

重定向：由浏览器自己再次访问另一个地址

![image-20200806182926304](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200806182926304.png)

转发：服务器将请求转发给另一个地址，浏览器不知道

![image-20200806183038924](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200806183038924.png)

## CSRF攻击

其他网站盗取cookie里的凭证向服务器提交表单，cookie存在本地容易被获取。

Security解决方案：在表单里生成一个隐藏的TOKEN，其他网站无法获取，TOKEN不会存在本地，在网络上传递，相对安全。

![image-20200807211050480](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200807211050480.png)

# 置顶、加精、删除

置顶：需要版主权限，修改帖子的类型，需要触发发帖事件，在ES服务器中更新帖子

加精：需要版主权限，修改帖子的状态，需要触发发帖事件，在ES服务器中更新帖子

删除：需要管理员权限，修改帖子的状态，需要触发删帖事件，在ES服务器中删除帖子

`thymeleaf-extras-springsecurity5`依赖包可以在前端结合Security判断用户权限

# Redis高级数据类型

## HyperLogLog

采用一种基数算法，用于完成独立总数的统计。

占据空间小，无论统计多少个数据，只占12K的内存空间。

不精确的统计算法，标准误差为0.81%。

## Bitmap

不是一种独立的数据结构，实际上就是字符串。

支持按位存取数据，可以将其看成是byte数组。

适合存储大量连续数据的布尔值（是否签到）。

## 网站数据统计

## UV（Unique Visitor）

使用HyperLogLog

独立访客，需通过用户IP（不论是否登录）去重来统计数据。

每次访问都要进行统计。

## DAU（Daily Active User）

使用Bitmap，以用户ID作为下标

日活跃用户，需通过用户ID去重来统计数据。

访问过一次，则认为其活跃。

# 任务执行和调度

JDK线程池：ExecutorService、ScheduledExecutorService

Spring线程池：ThreadPoolTaskExecutor、ThreadPoolTaskScheduler

- `@Async`注解让方法在多线程环境下，被异步调用。
- `@Scheduled(initialDelay = 10000, fixedRate = 1000)`注解让方法定时执行。

分布式定时任务：Spring Quartz，数据存储在数据库中，分布式服务器可以共享

- `Scheduler`核心调度接口。

- `Job`接口定义定时任务。
- `JobDetail`接口用于配置`Job`信息，参数会存在数据库的`qrtz_job_detai`表中。
- `Trigger`接口用于配置`Job`运行参数，参数会存在数据库的`qrtz_simple_trggers`和`qrtz_trggers`表中。

# 热贴排行

分数计算公式：
$$
log(精华分+评论分\times10+点赞数\times2+收藏数\times2)+(发布时间-网站建立时间)
$$
有分数变化操作时，将帖子id存入Redis，设置多线程定时任务计算分数，然后更新帖子分数同时更新ES服务器里面的数据。

# 生成长图

wkhtmltopdf：

- 下载安装wkhtmltopdf，配置环境变量。

- 新建存储路径。

- 生成PDF：`wkhtmltopdf url 存储路径` 

  生成图片： `wkhtmltoimage [--quality 75] url 存储路径`，`--quality`压缩图像

`Runtime.getRuntime().exec(cmd)`执行cmd命令。

生成事件，通过Kafka监听器异步生成。

# 优化网站性能

本地缓存

- 将数据缓存在应用服务器上，性能最好（空间小，无法跨服务器）
- 常用缓存工具：Ehcache、Guava、Caffeine

分布式缓存

- 将数据缓存在NoSQL上，**跨服务器**
- 常用缓存工具：MemCache、Redis等

多级缓存

- 一级缓存（本地缓存）——二级缓存（分布式缓存）——DB
- 避免缓存雪崩（缓存失效，大量请求直达DB），提高系统可用性

使用Caffeine缓存15条热贴

压力测试工具jmeter

# 单元测试

Spring Boot Testing：Junit、Spring Test、AssertJ等

测试用例：保证测试方法的独立性

常用注解：

@BeforeClass：测试类运行时执行一次

@AfterClass：测试类结束时执行一次

@Before：每次调用类中测试方法时执行一次

@After：每次结束类中测试方法时执行一次

# 项目部署

![image-20200809185157317](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200809185157317.png)

![image-20200809194854035](/images/JAVA%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/image-20200809194854035.png)