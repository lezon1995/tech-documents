# 阿里java架构师面试128题

# 一、Java基础和高级 

1.String类为什么是final的

> 因为字符串属于常量，常量是不允许被修改的，所以设计师在设计`String`类的时候，将String类设置为final

2.HashMap的源码，实现原理，底层结构。

3.反射中，Class.forName和classloader的区别

> `class.forName()`除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的`static`块，还会执行给静态变量赋值的`静态方法`
>
> `classLoader`只干一件事情，就是将.class文件加载到jvm中，不会执行`static`中的内容,只有在`newInstance`才会去执行`static`块

4.session和cookie的区别和联系，session的生命周期，多个服务部署时session管理。

> 第一次执行`request.getSession()`时创建，从不操作服务端资源开始30分钟之后session过期，或者服务端非正常关闭
>
> 或者调用`session.invalidate`手动关闭
>
> 正常关闭服务器后，session会持久化到磁盘上，等到下次服务端重新启动的时候，会加载到内存中

5.Java中的队列都有哪些，有什么区别。

> 非阻塞
>
> LinkedList	实现了`Deque`接口，是一个双端队列，可以当做普通队列使用
>
> ConcurrentLinkedQueue	实现了`Queue`接口，通过`CAS`机制来实现其`add`，`remove`方法，所以是线程安全的
>
> PriorityQueue	继承了`AbstractQueue`抽象类，通过比较元素的优先级，然后决定队列里面的元素顺序，元素需要实现`comparable`接口
>
> 阻塞
>
> ArrayBlockingQueue	有界阻塞队列
>
> LinkedBlockingQueue	无界阻塞队列
>
> PriorityBlockingQueue	带有优先级排序的无界阻塞队列
>
> SynchronousQueue	没有长度的阻塞队列，`put`直接阻塞，直到`take`被调用，`take`直接阻塞，直到`put`被调用
>
> DelayQueue	是一个无界的`BlockingQueue`，用于放置实现了`Delayed`接口的对象，其中的对象只能在其到期时才能从队列中取走，元素对象需要重写`compareTo`，`getDelay`两个方法

6.Java的内存模型以及GC算法

7.Java7、Java8的新特性

8.Java数组和链表两种结构的操作效率，在哪些情况下(从开头开始，从结尾开始，从中间开始)，哪些操作(插入，查找，删除)的效率高

9.Java内存泄露的问题调查定位：jmap，jstack的使用等等

> jmap 查看jvm中的堆中的信息
>
> jstack 查看jvm中的栈中的信息，主要用来排查线程问题

# 二、spring框架 

1.  spring框架中需要引用哪些jar包，以及这些jar包的用途

   > `core`	spring其他依赖中都需要的核心jar包
   >
   > `context`	spring的扩展包
   >
   > `beans`	spring管理bean的包
   >
   > `aop`	实现spring AOP功能相关的包
   >
   > `tx`	spring事务相关的包
   >
   > `dao`	数据库操作相关的包
   >
   > `jdbc`	spring对jdbc封装的包
   >
   > `web`	web开发时所需要用到的包
   >
   > `webmvc`	springmvc的相关包

2.  srpingMVC的原理

   ![img](https://img-blog.csdn.net/20180522162744760?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbndlaWhwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   1. 客户端（浏览器）发送请求，直接请求到`DispatcherServlet`。
   2. `DispatcherServlet`根据请求信息调用`HandlerMapping`，解析请求对应的`Handler`
   3. 解析到对应的`Handler`后，开始由`HandlerAdapter`适配器处理
   4. `HandlerAdapter`会根据`Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑
   5. 处理器处理完业务后，会返回一个`ModelAndView`对象，`Model`是返回的数据对象，`View`是个逻辑上的View
   6. `ViewResolver`会根据逻辑`View`查找实际的`View`
   7. `DispaterServlet`把返回的`Model`传给`View`
   8. 通过`View`返回给请求者（浏览器）

3.  springMVC注解的意思

4.  spring中beanFactory和ApplicationContext的联系和区别

   > ApplicationContext实现了beanFactory接口，实现了比beanFactory更丰富的管理bean的功能，和一些扩展功能，比如`国际化支持`，`资源访问`，`事件功能`
   >
   > beanFactory加载bean是延迟加载，这样会导致可能有些bean有问题，但是启动的时候并不能发现
   >
   > ApplicationContext是一次性全部加载所有的bean，这样就能很好的检查bean的完整性

5.  spring注入的几种方式

6.  spring如何实现事物管理的

7.  springIOC和AOP的原理

8.  spring中循环注入的方式

9.  Spring AOP与IOC的实现原理

10.  Spring的beanFactory和factoryBean的区别

    > `BeanFactory`是个Factory，也就是IOC容器或对象工厂
    >
    > `FactoryBean`是个Bean，也就是一个bean对象
    >
    > 在Spring中，所有的Bean都是由`BeanFactory`(也就是IOC容器)来进行管理的
    >
    > 但对`FactoryBean`而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean，它的实现与设计模式中的工厂模式和修饰器模式类似

11.  Spring的事务隔离级别，实现原理

12.  对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的？

13.  spring boot特性，优势，适用场景等

# 三、java多线程常见问题 

1.Java创建线程之后，直接调用start()方法和run()的区别

2.常用的线程池模式以及不同线程池的使用场景

3.newFixedThreadPool此种线程池如果线程数达到最大值后会怎么办，底层原理。

4.多线程之间通信的同步问题，synchronized锁的是对象，衍伸出和synchronized相关很多的具体问题，例如同一个类不同方法都有synchronized锁，一个对象是否可以同时访问。或者一个类的static构造方法加上synchronized之后的锁的影响。

5.了解可重入锁的含义，以及ReentrantLock 和synchronized的区别

6.同步的数据结构，例如concurrentHashMap的源码理解以及内部实现原理，为什么他是同步的且效率高

7.atomicinteger和volatile等线程安全操作的关键字的理解和使用

8.线程间通信，wait和notify

# 四、网络通信 

1.http是无状态通信，http的请求方式有哪些，可以自己定义新的请求方式么。

> get
>
> post
>
> put
>
> delete
>
> trace
>
> head
>
> option
>
> connect

2.socket通信，以及长连接，分包，连接异常断开的处理。

3.socket通信模型的使用，AIO和NIO。

4.socket框架netty的使用，以及NIO的实现原理，为什么是异步非阻塞。

5.同步和异步，阻塞和非阻塞。

# 五、常用Linux命令 

1.常用的linux下的命令

2.大的log文件中，统计异常出现的次数、排序，或者指定输出多少行多少列的内容。

3.linux下的调查问题思路：内存、CPU、句柄数、过滤、查找、模拟POST和GET请求等等场景

4.shell脚本

# 六、数据库MySql 

1.MySql的存储引擎的不同

2.单个索引、联合索引、主键索引

3.Mysql怎么分表，以及分表后如果想按条件分页查询怎么办(如果不是按分表字段来查询的话，几乎效率低下，无解)

4.分表之后想让一个id多个表是自增的，效率实现

5.MySql的主从实时备份同步的配置，以及原理(从库读主库的binlog)，读写分离

6.事物的四个特性，以及各自的特点（原子、隔离）等等，项目怎么解决这些问题

# 七、设计模式(写代码) 

1.单例模式：饱汉、饿汉。以及饿汉中的延迟加载

2.工厂模式、装饰者模式、观察者模式等

# 八、算法&数据结构&设计模式 

1.  使用随机算法产生一个数，要求把1-1000W之间这些数全部生成。（考察高效率，解决产生冲突的问题）
2.  两个有序数组的合并排序
3.  一个数组的倒序
4.  计算一个正整数的正平方根
5.  说白了就是常见的那些查找排序算法
6.  数组和链表数据结构描述，各自的时间复杂度
7.  二叉树遍历
8.  快速排序
9.  BTree相关的操作
10.  在工作中遇到过哪些设计模式，是如何应用的
11.  hash算法的有哪几种，优缺点，使用场景
12.  什么是一致性hash
13.  paxos算法

# 九、分布式缓存 

1.为什么用缓存，用过哪些缓存，redis和memcache的区别

2.redis的数据结构

3.redis的持久化方式，以及项目中用的哪种，为什么

4.redis集群的理解，怎么动态增加或者删除一个节点，而保证数据不丢失。（一致性哈希问题）

# 线程池、高并发、NIO 

1.  分析线程池的实现原理和线程的调度过程
2.  线程池如何调优
3.  线程池的最大线程数目根据什么确定
4.  动态代理的几种方式
5.  HashMap的并发问题
6.  了解LinkedHashMap的应用吗
7.  反射的原理，反射创建类实例的三种方式是什么？
8.  cloneable接口实现原理，浅拷贝or深拷贝
9.  Java NIO使用
10.  hashtable和hashmap的区别及实现原理，hashmap会问到数组索引，hash碰撞怎么解决
11.  arraylist和linkedlist区别及实现原理
12.  反射中，Class.forName和ClassLoader区别
13.  String，Stringbuffer，StringBuilder的区别？
14.  有没有可能2个不相等的对象有相同的hashcode
15.  简述NIO的最佳实践，比如netty，mina
16.  TreeMap的实现原理

# JVM相关(面试必考) 

1.  JVM内存分代
2.  Java 8的内存分代改进
3.  JVM垃圾回收机制，何时触发MinorGC等操作
4.  jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等
5.  你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1
6.  新生代和老生代的内存回收策略
7.  Eden和Survivor的比例分配等
8.  深入分析了Classloader，双亲委派机制
9.  JVM的编译优化
10.  对Java内存模型的理解，以及其在并发中的应用
11.  指令重排序，内存栅栏等
12.  OOM错误，stackoverflow错误，permgen space错误
13.  JVM常用参数

# 分布式相关 

1.  Dubbo的底层实现原理和机制

2.  描述一个服务从发布到被消费的详细过程

3.  分布式系统怎么做服务治理

4.  接口的幂等性的概念

5.  消息中间件如何解决消息丢失问题

   > 处理消息丢失的问题需要从三个方面考虑
   >
   > - `生产者`丢失
   >
   >   可能由于网络的原因，导致生产者发送的消费并没有被中间件收到，此时就需要开启消息的`事务机制`，通过在`生产端`调用`中间件`提供事务确认方法，如果收到了来自中间件的确认回应，那么就代表这条消息成功被中间件收到了
   >
   > - `中间件`丢失
   >
   >   假如`中间件`在收到消息之后，此时服务器宕机，导致消息丢失，这种情况下就需要开启消息中间件的`持久化`功能，可以配合上面的`事务确认机制`，在中间件持久化消息之后再给`生产者`返回一个确认信息。假如中间件宕机，重启之后也可以重新读取磁盘上的消息信息来达到恢复消息的目的
   >
   > - `消费者`丢失
   >
   >   假如在`中间件`给`消费者`发送消息之后，由于网络原因导致`消费者`并没有消费完这条消息，就可能导致这条消息丢失。解决方法是通过在`消费端`调用`中间件`提供的消息确认机制，当消息成功消费完之后，给中间件返回一个ACK确认，表示这条消息已经成功消费，在这之前，中间件会一直保留该消息，这样一来就可以全面保证消息不会丢失

6.  Dubbo的服务请求失败怎么处理

   > dubbo服务请求失败后，会尝试重新调用，在达到重试调用次数上限的时候会直接报错

7.  重连机制会不会造成错误

8.  对分布式事务的理解

   > `事务`这个词本来时出现在数据库中，代表一个事务中的所有操作都执行成功了才提交事务，只要其中出现一个操作有问题，那么立即结束事务并回滚事务。
   >
   > 早在系统架构还是单系统的时候，所有对数据库的操作都在一台机器里执行，仅仅调用数据库API就可以简单的开启事务，执行语句，以实现数据库的事务安全。
   >
   > 随着用户量增大，单系统慢慢支撑不了大量用户带来的并发量，于是系统升级为多系统架构，也就是多台机器，随着业务的复杂度提高，有时候会遇到需要调用多个服务的功能，但是这个功能是需要事务支持的，问题来了，在多系统中怎么实现对数据库的事务，此时事务从线程层面上升的进程层面，多台机器上的进程相互配合来实现的事务就叫分布式事务

9.  如何实现负载均衡，有哪些算法可以实现？

   > 轮询
   >
   > 随机
   >
   > 权重
   >
   > 最少活跃
   >
   > 哈希

10.  Zookeeper的用途，选举的原理是什么？

    > `zookeeper`是一个分布式服务协调的管理工具
    >
    > 优先检查`ZXID`。`ZXID`比较大的服务器优先作为`Leader`
    >
    > 如果`ZXID`相同，那么就比较`myid`。`myid`较大的服务器作为`Leader`服务器

11.  数据的垂直拆分水平拆分。

12.  zookeeper原理和适用场景

    > zookeeper可以用作
    >
    > 服务注册中心
    >
    > 服务发布订阅
    >
    > 命名服务
    >
    > 分布式锁
    >
    > 分布式队列
    >
    > 集群管理
    >
    > `Master`选举

13.  zookeeper watch机制

14.  redis/zk节点宕机如何处理

15.  分布式集群下如何做到唯一序列号

    > - 每次需要唯一号的时候就往一张自增表里面查一条数据，获取自增后的ID
    > - 使用时间加业务字段组合
    > - 使用Twitter开源的雪花算法

16.  如何做一个分布式锁

    > 使用redis的setNX的原子指令来创建一个key，如果创建成功说明获取了锁，创建失败说明没有获取锁，释放锁的时候将key删除，缺点假如创建锁之后，服务宕机，那么这个锁就永远存在，其他进程就永远无法获取锁，可以加过期时间来解决这个问题
    >
    > 使用zookeeper的临时有序节点，通过API创建临时有序节点，节点需要最小的那个获取锁，然后给比它大1的那个节点创建一个`Watch`监听器，然后创建了比序号大的锁的进程会阻塞，当最小节点被删除的时候，触发比它大1的那个节点的监听器，让其停止阻塞，就相当于获取了锁，后面的节点以此这么处理，上监听器。
    >
    > 

17.  用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗

    > ActiveMQ	万级吞吐量，实现AMQP协议，安全性最高，效率较慢，社区较为冷清
    >
    > RabbitMQ	万级吞吐量，社区活跃度还可以，但是由于erlang编写，java工程师不好去维护或者优化
    >
    > RocketMQ	十万级吞吐量，阿里团队实现的中间件，支持分布式，品质有保障
    >
    > Kafka	十万级吞吐量，由于其速度非常快，处理数据较小，天生适用于分布式环境，经常使用在大数据领域，用作数据收集，日志采集

18.  MQ系统的数据如何保证不丢失

19.  列举出你能想到的数据库分库分表策略；分库分表后，如何解决全表查询的问题。

# 数据库 

1.  MySQL InnoDB存储的文件结构

2.  索引树是如何维护的？

3.  数据库自增主键可能的问题

4.  MySQL的几种优化

5.  mysql索引为什么使用B+树

6.  数据库锁表的相关处理

7.  索引失效场景

8.  高并发下如何做到安全的修改同一行数据，乐观锁和悲观锁是什么，INNODB的行级锁有哪2种，解释其含义

   > 行锁分两种
   >
   > 共享锁（读锁）`lock in share mode`手动开启
   >
   > 排他锁（写锁）`insert`、`update`、`delete`默认自动开启，select可以使用`for update`手动开启
   >
   > 行锁的算法实现
   >
   > `记录锁record`
   >
   > `间隙锁gap`
   >
   > `临键锁next-key`
   >
   > 在mysql数据库中的repeatable read事务隔离级别下，临键锁next-key解决了幻读的问题

9.  数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁

# Redis&缓存相关 

1.  Redis的并发竞争问题如何解决了解Redis事务的CAS操作吗

   > multi: 开启一个事务，类比于mysql的openSession
   > exec: 提交一个事务，类比于mysql的commit
   > discard: 取消一个事务，类比于mysql的rollback
   > watch:监控一个key,与redis事务机制结合使用，形成原子锁

2.  缓存机器增删如何对系统影响最小，一致性哈希的实现

   > 使用一致性哈希算法来实现redis集群
   >
   > 增加的节点如果是新的key的最近顺时钟节点，就将key设置到新节点上
   >
   > 删除的节点会将节点上的key全部顺时钟转移到最近的节点上面

3.  Redis持久化的几种方式，优缺点是什么，怎么实现的

   > RDB
   >
   > AOF

4.  Redis的缓存失效策略

   > 定期删除
   >
   > 惰性删除

5.  缓存穿透的解决办法

   > 在缓存失效的第一个请求到来的时候加锁，让后面的请求CAS等待，或者阻塞
   >
   > 在去DB查询到数据之后重新加到缓存中，然后施放锁，后面的请求就会重新去缓中获取

6.  redis集群，高可用，原理

7.  mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

   > 设置内存淘汰策略为`allkeys-lru`或者`volatile-lru`，然后设置`maxmemory`为20G

8.  用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次

9.  redis的数据淘汰策略

   > 有6种
   >
   > noeviction
   >
   > allkeys-lru
   >
   > allkeys-random
   >
   > volatile-lru
   >
   > volatile-random
   >
   > volatile-ttl