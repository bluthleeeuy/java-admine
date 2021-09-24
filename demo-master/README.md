# java生产项目常用的demo

文档比较长，有兴趣查看的可以安装一个：
https://chrome.google.com/webstore/detail/github-toc/nalkpgbfaadkpckoadhlkihofnbhfhek
来展示目录

## 一.代理模式
### 1.静态代理
`继承方式实现` 
`聚合方式实现`
### 2.动态代理
`使用jdk proxy代理接口方式实现`
`使用Cglib代理类方式实现`
#### 自己实现的动态代理
`模仿jdk proxy自己实现动态代理`：
核心：实现动态代理的Proxy类 ，
实现动态代理的InvocationHandler类重写invoke方法实现.
## 二.spring
### 1.spring ioc
#### 1.1.spring容器
###### 1.1.1.BeanFactory
常用实现类：`DefaultListableBeanFactory`	
###### 1.1.2.ApplicationContext
常用实现类：`FileSystemXmlApplicationContext`
##### 1.1.3.启动过程
ioc容器的启动过程分为三个过程分别是：定位，载入，注册这三个基本过程<br>
将这三部分分开可以让用户更加灵活的对这三个过程进行裁剪或扩展，定义出最适合自己的ioc容器初始化过程。
由于对于不同容器启动过程是类似的，因此在基类`AbstractXmlApplicationContext`中将它们封装好，通过refresh()方法进行调用。
###### 定位
这个过程是BeanDefinition的定位，通过使用ResourceLoader的统一resource接口定位类似FileSystemResource和ClassPathResource等的Resource资源。<br>
###### 载入
这个过程是把用户定义好的Bean表示成Ioc容器内部的数据结构：BeanDefinition结构。<br>
###### 注册
向Ioc容器中注册BeanDefinition的过程，这个过程是调用BeanDefinitionRegistry接口的实现来完成的，在Ioc容器的内部通过将BeanDefinition注入到HashMap中去，Ioc容器就是通过这个HashMap来持有这些BeanDefinition数据的。<br>
在这个过程中，一般不包括bean的依赖注入。在Spring Ioc设计中Bean定义的载入和诸如是两个不同的过程。<br>
其中，依赖注入一般发生在第一次通过getBean向容器索取Bean的过程中。（但是有一个例外，当Bean设置了LazyInit属性，那么这个Bean的依赖注入在Ioc容器的初始化过程就完成了，而不用等到容器初始化以后的getBean方法）

###### refresh():
IOC容器初始化具体分为以下三部：
| BeanDefinition的Resource|定位|载入|注册|
| :-------- | --------:|--------:|--------:|
||对不同BeanDefinition Resource的定位|把用户定义的Bean表示成Ioc容器内部数据结构|调用BeanDefinitionRegistry注册BeanDefinition到IOC容器的HashMap中|
此时的ioc容器初始化过程一般`不包含Bean依赖注入`,一般依赖注入是在第一次getBean时才进行（设置了lazyinit的除外）。
###### registerBeanDefinition：
```java
DefaultListableBeanFactory:
	this.beanDefinitionMap.put(beanName, beanDefinition);
```
##### 1.1.4.ioc容器的依赖注入
```java
AbstractBeanFactory:
	protected <T> T doGetBean(
			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
			throws BeansException 
```
##### 1.1.5.Bean实例化
默认使用cglib对java的字节码进行增强。
`SimpleInstantiationStrategy`：
提供两种实例话方案，一种是通过BeanUtils，使用了`JVM反射`。
一种就是通过`cglib`。
##### 1.1.6.设置Bean对象的依赖：
```java
AbstractAutowireCapableBeanFactory:
populateBean
```
### 2.spring的事务
### 2.1.编程式事务
`作为例子，使用场景很少`
### 2.2.声明式事务
#### 使用spring-AspectJ（编译时实现动态代理）
`通过配置切点，切面织入实现动态代理`
#### 使用spring-AOP动态代理（运行时实现动态代理）
`注解方式（适合中小工程）`
#### 事务传播行为
#### 事务隔离级别

## 三.数据库事务
### 1.1数据库事务调优原则
在不影响业务应用的前提下:
1.`减少锁的覆盖范围（如：表锁->行锁）`<br>
2.`增加锁上可并行的线程数（如：读锁写锁分离，允许并行读取数据）`<br>
3.`选择正确锁的类型（如：悲观锁，乐观锁）`<br>
### 1.2JAVA中的事务
#### 1.2.1事务中的锁
##### sychronized
托管给JVM执行，采用CPU悲观锁的机制，获取的是线程独占锁，独占锁意味着其他线程只能依赖阻塞来等待线程释放锁，独占锁意味着其他线程只能依赖阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。<br>
##### Lock
Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就 是CAS操作（Compare and Swap）。<br>
##### 简单区别
synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。<br>
1.某个线程在等待一个锁的控制权的这段时间需要中断<br>
2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程<br>
3.具有公平锁功能，每个到来的线程都将排队等候、<br>
4.lock支持锁获取超时，而synchronized不支持。<br>
### 1.3 JAVA垃圾收集器
#### 基本概念
主流JVM虚拟机一般使用HotSpot，因此主要讨论HotSpot中的垃圾收集器。<br>
在理解java垃圾收集器之前需要明确一个概念：<br>
垃圾收集器主要分为三大类，一类是并行(Parallel)：这类收集器指很多垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。(属于并行并行的垃圾收集器有：ParNew，Parallel Scavenge,Parallel Old)<br>
另一类是并发(Concurrent)：指用户线程与垃圾收集器同时执行（但不一定会并行，可能会交替执行），用户程序在继续执行，而垃圾收集器运行于另一个CPU上。(属于并发的垃圾收集器有：CMS，G1)<br>
还有一种就属于简单串行（Serial）：指每次进行垃圾收集时，用户线程会处于等待，并且仅有一个线程能进行垃圾收集(Serial,Seral Old)。<br>
#### Serial收集器
这个收集器是一个单线程的收集器，它单线程的意义不仅仅说明它只会使用一个CPU或一条收集器线程去完成所有垃圾收集器工作，更重要的是它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。（Stop the world），这就意味着很可能你的程序运行每一小时要暂停5分钟。不过Java1.7以前Client模式下新生代默认的手机器就是它，因为在仅有一个CPU的情况下，使用其他并发收集器，并不是好的选择。
#### ParNew收集器
ParNew收集器其实就是serial收集器的多线程版本，除了使用多线程进行垃圾收集之外，其余行为包括serial收集器可用的所有参数，收集算法，stop the world，对象分配规则，回收策略等都和Serial收集器完全一样，在实现上也共用了相当多的代码。但是相比之下ParNew收集方式用的十分普遍其中一个重要的原因是：目前只有ParNew能与CMS配合工作。
#### Parallel Scavenge 收集器
与ParNew一样Parallel Scavenge也是使用复制算法同时也是并行的多线程收集器，它的特点是它关注的点与其他收集器不同，CMS等收集器的关注点是尽可能的缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量(ThroughtPut)。所谓吞吐量就是CPU运用用户代码的时间与CPU总消耗时间的比值。<br>
停顿时间越短的越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可以高效的利用CPU时间，尽快完成计算任务，主要适合后台运算而交互不太多的任务。<br>
其中Parallel Scavenge收集器提供了两个参数用于精确控制吞吐量（-XX:MaxCGPauseMills最大垃圾收集停顿时间，-XX:GCTimeRatio直接设置吞吐量大小）另外也经常使用参数（-XX:+UseAdaptiveSizePolicy,当这个开关打开后就不需要手动指定新生代大小，晋升老年代老年代年龄等），虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大吞吐量，这种调节方式称为GC自适应调节策略。
#### Serial Old收集器
使用标记整理算法的老年代单线程收集器。
#### Parallel Old收集器
Parallel是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法。在注重吞吐量遗迹CPU资源敏感的场合，可以优先考虑Parallel Scavenge加Parallel Old收集器。
#### CMS收集器
目前很大一部分JAVA应用集中在互联网或B/S系统的服务端，这类应用尤其注重响应速度，希望系统能停顿时间越短越好，而CMS收集器就非常符合。<br>
CMS(Concurrent Mark Sweep)是一种基于“标记－清除”算法实现的，它的运作过程相对于前面几种更加复杂。<br>
初始标记()CMS initial mark)<br>
并发标记(CMS concurrent mark)<br>
重新标记(CMS remark)<br>
并发清除(CMS concurrent sweep)<br>
其中初始标记仅仅标记一下GC Roots能直接关联到的对象。速度很快，并发标记阶段就是GC RootTracing过程，而重新标记阶段则是为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记纪录，这个阶段的停顿时间会比初始标记长一点，但是要远小于并发标记。<br>
由于整个过程中最耗时的并发标记和并发清楚都是垃圾收集线程和用户线程一起并发执行的，因此停顿时间会很短。<br>
但CMS还远远做不到完美，还存在有一下一些缺点：<br>
1.CMS收集器对CPU资源非常敏感，其实面向并发设计的程序都对CPU资源比较敏感。在并发阶段它虽然不会导致用户线程停顿，但是会因为占用了一部分资源，导致吞吐量降低。CMS默认的垃圾回收线程是（CPU数量＋3）／4，也就是说当CPU资源负载比较大时，吞吐量会进一步下降产生更大压力。<br>
2.CMS无法处理浮动垃圾(由于在CMS进行垃圾清理的过程中用户线程也在运行，此时产生的垃圾就是浮动垃圾)，可能出现Concurrent Mode Failure 失败而导致另一次Full GC产生。而CMS需要预留一部分空间提供并发收集时的程序运作使用（JDK1.5这部分比例为大于68％就会触发CMS收集，JDK1.6提升到了92%）。要是CMS运行期间预存的内存无法满足程序需要，就会出现Concurrent Mode Failure。此时就会启动临时预案，使用Serial Old收集器来重新进行老年代的垃圾收集。所以-XX:CMSInitiatingOccupancyFraction设置太高会导致大量Concurrent Mode Failure失败，性能反而变低。<br>
3.还有一个缺点是CMS是基于标记清除算法的，这就意味着收集结束时会产生很多碎片，空间碎片过多时，将会给大对象分配造成苦难，提早产生Full GC，而CMS提供了参数-XX:UseCMSConcurrentAtFullConllection开关（默认启动），用于在CMS收集器快要进行Full GC时，进行一次内存碎片的压缩操作，但这个操作不是并发的，因此会有停顿。还有一个操作－XX:CMSFullGCsBeforeCompaction,这个参数是用于设置执行多少次不压缩FullGC后进行一次压缩(默认是0)。
#### G1收集器
##### `G1有几大特征`：
1.并行与并发:G1能充分利用多CPU，多核环境下的硬件优势，使用多个CPU来缩短Stop the world停顿时间，部分其他收集器原本需要停顿Java线程执行GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。<br>
2.分代收集：与其他收集器一样，分代收集的概念在G1中依然得以保存。虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但它能够采用不同的方式去处理新创建的对象和已经存活了一段时间，熬过多次GC的就对象以获取更好的收集效果。<br>
3.空间整合：与CMS的“标记－清除”算法不同，G1从整体来看是基于“标记－整理”算法实现的收集器，从局部（两个Region之间）上来看是基于“复制”算法实现的。但无论如何这两种算法都意味着G1运作期间不会缠哼内存空间碎片，不会因为碎片空间导致分配大对象而进行一次GC。<br>
4.可预测停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同关注的点，但是G1处了追求停顿外，还能建立可预测的停顿时间模型，还能让使用者明确制定在一个长度为M浩渺的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。<br>
##### `G1的做法`：
它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不在是物理隔离的了。他们都是Region的一部分。G1之所以能能建立可预测的停顿时间模型，是因为它可以有计划的避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆的价值大小，在后台维护一个优先级列表，每次根据允许的收集时间，优先回收价值最大的Region（这也是Garbage-First名字的由来）.这种方式保证了G1收集器在有限时间内可以获取尽可能高的收集效率.而对于不同Region引用了相同对象通过使用Rememvered Set来避免全堆扫描。
#### `具体回收步骤`：
如果不计算维护Remembered Set的操作，G1收集器的运作大致可划分为以下几个步骤：<br>
初始标记(Initial Marking)<br>
并发标记(Concurrent Marking)<br>
最终标记(Final Marking)<br>
筛选回收(Live Data Counting and Evacation)<br>
### 1.4 JAVA NIO
Java NIO提供了与标准IO不同的IO工作方式：<br>
`Channels and Buffers（通道和缓冲区）：`标准的`IO`基于`字节流`和`字符流`进行操作的，而`NIO`是基于`通道（Channel）和缓冲区（Buffer）`进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。<br>
`Asynchronous IO（异步IO）：`Java NIO可以让你`异步`的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。<br>
`Selectors（选择器）：`Java NIO引入了`选择器`的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。<br>
`Channel:`基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。<br>
常见的channel实现有:<br>
FileChannel<br>
DatagramChannel<br>
SocketChannel<br>
ServerSocketChannel<br>
这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。 <br>
`Buffer:`实现覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char,Java NIO 还有个 Mappedyteuffer，用于表示内存映射文件。 <br>
`Selector:`Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。 <br>
要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。 
<br>
#### NIO与IO的区别
1.面向流与面向缓冲:<br>
 `Java IO`面向流意味着每次从流中`读一个或多个字节`，直至读取所有字节，它们没有被缓存在任何地方。此外，它`不能前后移动流中的数据`。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 `Java NIO`的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时`可在缓冲区中前后移动`。这就增加了处理过程中的灵活性。但是，还`需要检查是否该缓冲区中包含所有您需要处理的数据`。而且，需确保当更多的数据读入缓冲区时，`不要覆盖缓冲区里尚未处理的数据`。 <br>
 2.阻塞与非阻塞IO:<br>
 `Java IO`的各种流是`阻塞`的。这意味着，当一个线程调用`read() 或 write()时，该线程被阻塞`，直到有一些`数据被读取，或数据完全写入`。该线程在此期间不能再干任何事情了。 `Java NIO的非阻塞模式`，使一个线程从某通道发送请求读取数据，`但是它仅能得到目前可用的数据`，如果目前没有数据可用时，就什么都不会获取。而`不是保持线程阻塞`，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 `线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作`，`所以一个单独的线程现在可以管理多个输入和输出通道（channel）`。 <br>
 3.Selector:<br>
 Java NIO的选择器允许一个`单独的线程来监视多个输入通道`，你可以`注册多个通道使用一个选择器`，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。 
 `总结：`NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。 <br>
`NIO使用场景：`如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。<br>
`IO使用场景：`如果你有少量的连接使用非常高的带宽，一次发送大量的数据，也许典型的IO服务器实现可能非常契合。<br>




## 四.简单应用
### 1.1 高并发秒杀系统（持续更新）
项目目录：https://github.com/asdgh1000/demo/tree/master/secoundKill
### 1.2 简单代理模式demo（持续更新）
项目目录：https://github.com/asdgh1000/demo/tree/master/proxyDemo
### 1.3 简单spring事务demo（持续更新）
项目目录：https://github.com/asdgh1000/demo/tree/master/springTrancation
## 五.分布式
1.分布式协作配置管理系统（zookeeper）<br>
2.分布式缓存系统<br>
3.分布式持久化存储<br>
4.分布式消息系统<br>
5.搜索引擎<br>
6.CDN<br>
7.负载均衡系统<br>
8.运维自动化系统<br>
9.实时计算系统<br>
10.离线计算系统<br>
11.分布式文件系统<br>
12.日志收集系统<br>
13.监控系统<br>
14.数据仓库<br>
### 难点
1.缺乏全局时钟<br>
2.面对故障独立性<br>
3.处理单点故障<br>
4.事务的挑战<br>
### 1.1 分布式相关算法
#### 1.1.1 负载均衡算法
##### 1.Round Robin(轮询算法)
存在问题：轮询策略的目的在于，希望做到请求转移的完全均衡，但为了pos变量修改的互斥性，需要引入重量悲观锁，sychronized，将会导致该轮询代码的并发吞吐量明显下降。
##### 2.Random (随机算法)
当请求量大的时候，接近于平均访问，并且不需要使用悲观锁，吞吐量比轮询算法高，但是可能会出现局部数据过热等问题。
##### 3.Hash (源地址hash)
源地址哈希的思想是获取客户端访问Ip地址值，经过哈希计算得到一个值，用该值对服务器列表大小进行取模运算，得到的结果便是要访问的服务器序号。<br>
优势：同意客户端Ip，当后端服务器列表不变时，它每次都会被映射到同一台后端服务器访问。（根据此特征，可以在服务消费者与服务提供者之间建立有状态的session会话）。
##### 4.Weight Round Ronbin (加权轮询法)
##### 5.Weight Random (加权随机法)	
加权可以防止机器的抗压能力不同造成负载问题。
##### 6.Least Connections (最小连接数法)
####1.1.2 一致性hash算法
例如使用源地址ip Hash来进行负载均衡，通常只使用Hash算法会造成若一个机器挂掉，大量的数据会因为`hash算法`而进行节点之间的迁移，会造成很高的`成本开销`。所以使用`一致性hash`来避免这种问题。<br>
一致性哈希所带来的最大变化是把节点对应的哈希值变成了一个范围，而不再是离散的。在一致性哈希中，整个哈希值的范围定义非常大，然后把这个范围分配给现有节点。<br>
##### 一般的一致性哈希算法
如果有节点加入，那么这个新节点会从原有的某个节点上分管一部分范围哈希值；<br>
如果有节点退出，那么这个节点原本管理的哈希值会给它的下一个节点管理。<br>
###### 存在问题
每当新增一个节点，除了新增节点外，只有一个节点受影响，这个新增节点和受影响的节点负载是明显比其他节点低的。<br>
减少一个节点时，除了减去的节点外，只有一个节点受到影响，它要承担自己原来的和减去的节点的工作，压力要高于其它节点。似乎要增加一倍的节点或减去一倍的节点才能保持各个节点的负载均衡。
##### 虚拟节点对一致性哈希的改进
引入虚拟节点，即4个物理节点可以变为很多虚拟节点，每个虚拟节点支持连续哈希环上的一段。<br>
这时如果加入一个物理节点，就会相应加入很多虚拟节点。这些虚拟节点是相对均匀的插入到整个哈希环上的。这样就能够很好的分担现有物理节点的压力。如果减少一个物理节点，对应的很多虚拟节点就会失效，这样就会有很多剩余的虚拟节点承担之前虚拟节点的工作，胆识对于物理节点来说，增加的负载相对是均衡的。所以通过一个物理节点对应增加很多的虚拟节点，并且同一个物理节点的虚拟节点尽量均匀分布的方式来解决增加或减少节点时负载不均匀的问题。


### 1.2 分布式系统间通讯
#### 1 java 自身技术实现消息方式的系统间通讯
##### 1 基于消息方式实现系统间通讯
###### 1 TCP/IP+BIO
###### 2 TCP/IP+NIO
###### 3 TCP/IP+AIO
##### 2 开源框架实现系统间通讯
###### 1 Mina

#### 2 基于远程调用方式实现系统间通讯
##### 1 RMI(Remote Method Invocation)
客户端只有服务端提供的接口，通过此接口实现对远程服务端的调用。
##### 2 webService
##### 3 其他开源框架
###### 1 spring RMI
###### 2 CXF 
### 1.3序列化方案
#### 1 java内置序列化
#### 2 hessian 序列化方式
代码地址:<br>
https://github.com/asdgh1000/demo/tree/master/serialize-test/serializeMethod
### 1.4 简单分布式事务实践（持续更新）
#### 1 XA 
XA是由X/Open组织提出的分布式事务的规范。XA规范主要定义了（全局）事务管理器（Transaction Manager）和（局部）资源管理器（Resource Manager）之间的接口。<br>
XA接口是双向的系统接口，在事务管理器（Transaction Manager）以及一个或多个资源管理器（Resource Manager）之间形成通讯桥梁。<br>
XA之所以需要引入事务管理器是因为：在分布式系统中，从理论上讲，两台机器理论上无法达到一致的状态，需要引入一个单点进行协调。<br>
事务管理器控制着全局事务，管理事务生命周期，并协调资源。资源管理器负责控制和管理实际资源（如数据库或JMS队列）。<br>
#### DTP (Distributed Transaction Processing Reference Model) 分布式事务处理模型
##### (Application Program) AP
应用程序：<br>
可以理解为使用DTP模型的程序，它定义了事务边界，并定义了构成该事务的应用程序的特定操作。<br>
AP和
##### (Resource Manager) RM
资源管理器：<br>
可以理解为DBMS系统，或者消息服务管理系统。应用程序通过资源管理器对资源进行控制，资源必须实现XA定义的接口，资源管理器提供了存储共享资源支持。<br>
##### (Transaction Manager) TM
事务管理器：<br>
负责协调和管理事务，提供给Ap应用程序编程接口并管理资源管理器。事务管理器向事务指定标识，监听它们的进程，并负责处理事务的完成和失败。<br>
######  XID
事务分支识别：<br>
由TM指定，以标识一个RM内的全局事务和特定分支。它是TM中日志与RM中日志之间的相关标记。两阶段提交或回滚需要XID，以便在系统启动时执行再同步操作（也称为再同步（resync）），或在需要时允许管理员执行试探操作（也成为手工干预）。
##### 事务
事务的本质是：`锁` 和 `并发` 
事务的最大作用是保证 `一致性`
`ACID` 保证事务的`完整性`
###### 事务单元
一个事务是一个完整的工作单元，由多个独立的计算任务组成，这多个任务在逻辑上是原子的。(所有的真对一个数据的操作 都可以认为是事务)<br>
例如这些都是事务单元：<br>
商品要创建一个基于GMT_Modified的索引<br>
从数据库中读取一行纪录<br>
向数据库中写入一行纪录，并同时更新这行纪录的所有索引<br>
删除一整张表<br>等等。。。
####### 事务单元Happen-before关系
`读写`,`写读`,`读读`,`写写`
原子性（Atomicity）:事务要么成功执行（提交），要么所有的动作都不发生（回滚）。<br>
一致性(Consistency）：事务产生一致的结果并且保证程序的完整性约束。（核心：保证一个事务单元全部成功后才可见）<br>
隔离性（Isolation）：事务执行时的中间状态对其它事务是不可见的。此外事务应该表现为连续执行，即使实际是并发执行的。<br>
持久性（Durability）：提交事务的结果是永久性的（即使是灾难性的故障）<br>

事务隔离级别（事实上`隔离性`是对一致性的破坏:因为理论上来说最好的能保证一致性的方式是串行的执行所有读写请求，而隔离性出现让不同操作之间可以并行执行，这样不同的隔离级别会产生不同的一致性状态，这么做的原因也是在一致性和性能之间做的权衡（`以性能为理由对一致性的破坏`））：<br>
（随着隔离级别的升高死锁的可能性也变大）<br>
sql92标准制定的事务隔离级别：<br>
序列化：对应的是 `排他锁`<br>
可重复读：只能做到读和读之间并行（因为在读读并行时若是有写，则无法读并行会造成不可重复读） 对应的是 `同一个事物内读写锁，且锁不可升级（读读不加锁）即：读时不能写数据，写时更不能读`<br>
读已提交：只能读取到写事务已经提交的数据 对应的是 `同一个事物内读锁可升级为写锁，读之间没有锁`<br>
读未提交：能读到写事务还未提交的数据 对应的是 `只有写锁` <br>

###### 隔离性扩展：MVCC（快找隔离级别）
本质：copyOnWrite 无锁编程<br>
对写读的场景进行优化 每次写时都是创建副本来写，读不受影响
最大的好处是：保证 `读写` `写读并行`
##### 锁
###### 锁的并发级别
####### 阻塞
一个线程是阻塞的，那么在其他线程释放资源之前，当前线程无法继续执行。当我们使用synchronized关键字，或者重入锁时，我们得到的就是阻塞线程。无论是synchronized或重入锁，都会试图在执行后续代码前得到临界区锁，如果得不到线程就会被挂起等待，直到占有了所有资源为主。<br>
####### 无饥饿
使用公平锁可以避免线程饥饿。
####### 无障碍
无障碍是一种最弱的非阻塞调度。两个线程如果是无障碍的执行，那么他们不会因为临界区的问题导致一方被挂起。如果大家一起修改数据，对于无障碍的线程来说，一旦检测到这种情况，它就会立即对自己所做的修改进行回滚，确保数据安全。但如果没有数据竞争发生，那么线程就可以顺利的完成自己的工作，走出临界区。<br>
如果说阻塞的控制方式是悲观策略，无障碍非阻塞就是一种乐观策略。但一旦冲突比较多，这种无障碍方式回滚会比较多就不是很适合了。（同样也可以使用一致性标记实现无障碍的非阻塞调度）。<br>
#######无锁
无锁的并行就是无障碍的，在无锁的情况下，所有的线程都能尝试对临界区进行访问，但不同的是无锁的并发保证必然有一个线程能够在有限步骤完成操作离开临界区。<br>
####### 无等待
无锁只要求一个线程可以在有限步内完成操作，而无等待则在无锁的基础上更进一步进行扩展。它要求所有的线程都必须在有限步内完成，这样就不会引起饥饿问题。如果限制这个步骤上限，还可以进一步分解为有界无等待和线程数无关的无等待几种，他们之间的区别只是对循环次数的限制不同。<br>
######使用锁会出现的问题
####### 死锁
####### 饥饿
饥饿是指一个或者多个线程因为种种原因无法获得想要的资源，导致一直无法执行。（比如它的优先级可能太低，而优先级高的线程不断抢占它需要的资源，另外有可能的是，某一个线程一直占着关键资源不放，导致其他需要这个资源的线程无法正常执行）<br>
####### 活锁
如果线程设计的不合理，且都秉承着谦让的原则，主动将资源释放给他人使用，那么就会出现资源不断在两个线程中跳动，而没有一个线程可以同时拿到所有资源而正常执行。<br>
###### Java虚拟机对锁的优化
####### 锁偏向
锁偏向是一种针对加锁操作的优化手段。当一个线程得到了锁，那么锁就进入偏向模式。当这个线程再次请求锁时，无须再做任何同步操作。这样就节省了大量有关锁申请操作，从而提高了程序性能。因此在几乎没有锁竞争的场合，偏向锁有比较好的优化效果，因为连续多次极有可能使用一个线程请求相同的锁。但是对于竞争比较激烈的场合，效果不佳。因为在竞争激烈的场合，最有可能的情况是每次都是不同的线程来请求相同的锁。这样偏向模式会失败，因此还不如不启用偏向锁。使用JAVA虚拟机参数－XX:+UseBiasedLocking可以开启偏向锁<br>
####### 轻量级锁
如果偏向锁失败，虚拟机不会立即挂起线程。它还会使用一种称为轻量级锁的优化手段。轻量级锁的操作也很轻便，它只是简单的将对象头做为指针，指向持有锁的线程堆栈的内部，来判断一个线程是否持有对象锁。如果线程成功获得了轻量级锁，则可以顺利进入临界区。如果轻量级锁加锁失败，则表示其他线程抢先争夺到了锁，那么当前线程的锁请求就会膨胀位重量级锁。<br>
####### 自旋锁
膨胀后，虚拟机为了避免线程真实的在操作系统层面挂起，虚拟机还会在做最后的努力－－自旋锁。由于当前线程暂时无法获得锁，但是什么时候可以获得锁是一个未知数。也许在几个CPU时钟周期后，就可以得到锁。如果这样，简单粗暴地挂起线程可能是一种得不偿失的操作。因此，系统会进行一次赌注：它会假设在不久的将来，线程可以得到这把锁。因此，虚拟机会让当前线程做几个空循环（自旋的含义），在经过若干次循环后，如果可以得到锁，那么就顺利进入临界区。如果还不能得到锁，才会真实的将线程在操作系统层面挂起。
####### 锁消除
锁消除容易被人忽略但是它是非常实用而且有必要的优化，了解消除锁的原理对我们平时代码的优化有很大作用：<br>
java虚拟机在JIT编译时，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁。通过锁消除可以节省毫无意义的请求锁时间。<br>
可以看出来锁消除的关键点有两个 一个是虚拟机对不可能存在竞争的锁的消除，另一个就是如何判断该资源是不会被竞争的。举个例子：你很有可能在一个不可能存在并发竞争的场合下使用vector。众所周知，vector内部使用了stnchronized请求锁。
```java
	
	public String[] createStrings(){
		Vector<String> v = new Vector<String>();
		for(int i=0;i<100;i++){
			v.add(Integer.toString(i));
		}
		return v.toArray(new String[]{});
	}

```
由于变量v只在createStrings()函数中使用，因此，它只是一个单纯的局部变量。局部变量是在线程栈上分配的属于线程私有的数据，因此不可能被其他线程访问。所以在这种情况下，Vector内部所有加锁同步都是没有必要的。如果虚拟机检测到这种情况就会将这些无用的锁操作去掉。<br>
锁取消涉及一项关键技术为逃逸分析。所谓逃逸分析就是观察某一个变量是否会逃出某个作用于，在上面例子中v没有逃出createStrings()，以此为基础虚拟机才大胆的将v内部的加锁操作去除。如果createStrings（）返回的不是Stirng数组，而是v本身，那么就认为变量v逃逸出了当前函数，也就是说v有可能被其他线程访问。这样，虚拟机就不能消除v中的锁操作。<br>
逃逸分析必须在－server模式下进行，可以使用－XX:+DoEscapeAnalysis参数打开逃逸分析。使用-XX:EliminateLocks参数可以打开锁消除。

##### 全局事务
一次性操作多个资源管理器的事务就是全局事务。
##### 分支事务
在全局事务中，每一个资源管理器有自己独立的任务，这些任务的集合是这个资源管理器的分支事务。
##### 控制线程
用来表示一个工作线程，主要是关联AP，TM和RM这三者的线程，也就是事务上下文环境，简单的说就是用来标识全局事务和分支事务关系的线程。

### 1.5 分布式一致性
####2PC (ZK执行事务阶段是2PC的改良)
两阶段提交顾名思义是两阶段的，第一阶段完成提交事务请求，第二阶段负责执行事务提交。
##### 第一阶段：完成提交事务请求
1.事务询问：<br>
协调者向所有参与者发送事务内容，询问是否可以进行提交操作，并开始等待各参与者的响应。<br>
2.执行事务:<br>
各参与者节点执行事务，并将Undo和redo日志信息记入事务日志。<br>
3.各参与者向协调者反馈事务询问的响应：<br>
如果参与者成功执行了事务操作，那么就反馈给协调者Yes，否则No。（这边需要所有的参与者都反回Yes才会进行事务提交操作，而ZK并不是这样，只要大多数节点进行了Yes返回，就开始执行事务提交操作。然后通过propsal来保障事务提交的正确性，例如此事有节点断开，恢复时候会根据propsal记录需要补偿多少undo日志的提交）。<br>
##### 第二阶段：执行事务执行
1.发送提交请求：<br>
假如协调者从所有参与者处得到的反馈都是Yes，那么就会执行事务提交。
协调者向所有参与者发送commit请求。<br>
2.事务提交：<br>
参与者收到Commit后会正式执行事务提交，并在完成提交之后释放在整个事务执行阶段占用的资源。<br>
3.反馈事务提交的结果：<br>
参与者在完成事务提交之后，向协调者发送Ack消息。<br>
4.完成事务:<br>
协调者接收到所有参与者的Ack消息后，完成事务。(ZK中选择当大多数节点完成Ack消息反馈后就完成事务)<br>
##### 可能出现的中断：中断事务
假如任何一个参与者在投票阶段向协调者返回No，或者在超时之后，协调者尚无法接收到所有参与者的反馈响应，那么就会中断事务。<br>
中断过程：<br>
1.发送回滚请求：<br>
协调者向参与者节点发送RollBack请求。<br>
2.事务回滚:<br>
参与者在接到事务回滚的请求时会利用阶段一纪录的Undo日志进行事务的回滚，并在回滚完成后释放所有占用的资源。<br>
3.反馈事务回滚结果:<br>
参与者在完成事务的回滚之后，向协调者发送Ack。<br>
4。协调者在收到所有参与者的Ack后完成事务中断。<br>
要注意二阶段提交对每个事务都是先尝试后提交因此可以看作是一个强一致性算法。<br>
##### 二阶段提交存在的问题：
同步阻塞，单点问题，脑裂，太过保守。<br>
1.同步阻塞:<br>
在二阶段提交的执行过程中，所有参与该事务操作的逻辑都处在阻塞状态，也就是说，各个参与者在等待其他参与者响应的过程中，将无法进行其他操作。<br>
2.单点问题:<br>
一旦协调者出现问题，整个二阶段提交就无法运行。更严重的是，如果协调者在第二阶段发生错误，可能造成参与者一直处于锁定事务资源的状态。<br>
3.脑裂(数据不一致):<br>
若第二阶段发生Commit过程，发生了局部网络瘫痪，使得只有一部分机器收到了commit请求，会造成数据不一致情况。（3PC不存在，因为会提前询问）。
<br>
4.太过保守:<br>
协调者在进行提交事务询问的过程中，参与者出现故障而导致协调者始终无法获得参与者的反馈，就要一直等超时，只能等待中断机制，也就是说，二阶段提交没有设计完善的容错机制。任意一个节点的失败都会导致整个事务执行的失败。
<br>
通过2PC的情况可以理解成 2pc是保证AP 而放弃了C（高可用）.
##### 引入问题
1.由于事务管理器自身的稳定性，可用性的影响，以及网络通讯中可能产生的问题，出现的情况会很多。<br>
2.事务管理器在多个资源之间进行协调，它自身要进行很多日志记录工作。<br>
3.网络上的交互次数的增多以及引入事务管理器的开销，是使用两阶段提交协议使分布式事务的开销增大的两个方面。<br>
因此在进行垂直拆分或水平拆分后，需要想清楚是否一定要引入两阶段提交的分布式事务，有必要才使用。<br>
####3PC
三阶段提交顾名思义是三个步骤实现的分别是：canCommit,preCommit,doCommit<br>
##### 第一阶段：canCommit:
1.事务询问：<br>
协调者向所有参与者发送一条canCommit指令，询问是否可以执行事务提交操作，然后等待反馈。
2.各参与者向协调者反馈询问结果:<br>
参与者如果在接受到协调者的canCommit询问请求后，正常情况下，如果其自身认为可以顺利执行事务，那么反馈yes响应，并进入预备状态，否则返回no。<br>
##### 第二阶段：preCommit：
第二阶段会根据第一阶段反馈的结果进行事务的preCommit。<br>
可能一：执行事务预提交：<br>
当所有参与者都反馈yes，进入预提交。与2PC的第一阶段相同，执行事务，并记录undo redo日志，反馈Yes等待Commit请求。<br>
可能二:当有人和一个参与者反馈了No则，或者在等待超时之后，协调者尚无法接收到所有参与者的反馈响应，那么就会中断事务。<br>
发送中断请求，参与者不执行事务。
##### 第三阶段：doCommit:
该阶段同样存在两种可能，执行提交：<br>
一种是接收到了所有参与者的Ack响应，那么它将从preCommit 进入到 doCommit阶段，向所有参与者发送commit命令。而参与者进行事务提交，释放资源并反馈提交情况。协调者在收到所有的参与者的反馈后完成事务。<br>
中断事务:<br>
假设协调者没有故障，并且有任意一个参与者发送No响应，或者在超时后，仍然有参与者没有反馈信息，那么就会中断事务。协调者发送abort请求，参与者根据undo日志进行回滚，释放资源，并反馈事务回滚结果，在接到所有参与者的ACK操作后，中断事务。<br>
##### 与2PC的区别：
`一旦进入阶段三，无论是协调者出现问题，协调者与参与者同时出现问题，都会导致参与者最终无法收到协调者的doCommit或是abort请求，针对这种情况，参与者都会在等待超时后，继续执行事务提交（因为之前已经canCommit进行过投票）.`
###### 优点
降低了参与者的阻塞范围，并且能够出现单点故障后继续达成一致。
###### 缺点
在参与者接受到preCommit消息后，如果网络出现分区，此时协调者与参与者无法进行通讯，在这样的情况下该参与者仍然会进行事务提交。这就会导致事务不一致。

####Basic Paxos
#### 最终一致
#### CAP
##### C(Consistency) 数据一致性
当数据写入成功后，所有的节点会同时看到这个新的数据。
##### A(Availability) 数据可用性
保证无论是成功还是失败，每个请求都能收到一个反馈，重点是系统一定要有响应。
##### P(Partition-Tolerance) 容错性
即使系统中有部分问题或者消息丢失，但系统仍然能够继续运行，也就是系统的一部分出现问题时，系统仍能继续工作。
##### 存在问题
CAP三个属性是不能同时完全满足的，通常来说需要进行权衡。<br>
1.选择CA:放弃分区容忍性，加强一致性和可用性。这其实就是传统单机数据库的选择。<br>
2.选择AP:放弃一致性，追求分区容忍性及可用性。这是很多分布式系统在设计时的选择，例如很多NoSql就是如此。<br> 
3.选择CP:放弃可用性，追求一致性和分区容忍性。这种选择下的可用性会比较低，网络的问题会直接让整个系统不可用。<br>
#### BASE
##### Basically Available 基本可用
允许分区失败
##### Soft state 软状态
接受一段时间内的状态不同步
##### Eventtually consistent 最终一致性
保证最终数据状态是一致的
#### Quorum
N:数据复制节点数量<br>
R:成功读操作的最小节点数<br>
W:成功写操作的最小节点数<br>
如果W+R>N是可以保证强一致性的，如果W+R<=N，是能保证最终一致性的.<br>
根据CAP理论，需要在一致性，可用性，容错性方面进行权衡。例如让W=N且R=1，就会大大降低可用性，但是一致性是最好的.<br>
#### Vector Clock
Vector Clock的思路是对同一份数据的每一次修改都加上<修改者，版本号>这样的一个信息，用于记录修改者的信息及版本号。<br>
算法为每一个商议结果附上一个时间戳，当结果改变时，更新时间戳。<br>

###1.6 分布式调度框架:zk
zookeeper核心其实类似一个精简的文件系统。
#### zk基本使用：
Apache开源分布式服务框架，可以使用zk来实现分布式架构中的：<br>
1.数据发布／订阅<br>
2.负载均衡<br>
3.命名服务<br>
4.分布式协调通知<br>
5.master选举<br>
#### zk的典型应用场景及实现
`首先要说的是：Zk是解决分布式一致性问题的利器。`
##### Zk的使用场景一：数据发布／订阅
数据发布／订阅系统，即所谓的配置中心，发布者将数据发布到Zookeeper的一个或多个节点上，供订阅者进行数据订阅，从而达到动态获取数据的目的，实现配置信息的集中式管理和数据的动态更新。<br>
`方式：`Zk采用的客户端，服务器推拉结合的方式：客户端向服务器注册自己需要关注的节点，一旦节点数据发生变更，那么服务端就会向相应的客户端发送watcher事件通知，客户端接收到这个消息后，需要主动的向服务端获取新的数据。
###### 为什么要用ZK 不用其他方案？
在一般的系统中，经常会遇到这样的情况：系统需要一些通用的配置信息，这些信息通常：1.数据量比较小，2.数据内容在运行时会发生变化，3.集群中各机器共享，配置一致。对于这样的配置信息一般会选择使用将其`存储在本地配置文件或事内存变量中（JMX）`但是，机器规模一旦变大配置信息变更越来越频繁后，使用这两种方式变的越来越困难。因此需要寻找一种更为分布式化的解决方案。<br>
`具体做法：`就是创建Zk的节点，将配置信息写进去，并且让客户端在该配置节点上注册一个数据变更的watcher监听，一旦节点发生数据变更，所有订阅的客户端都能感知到数据变更。然后让客户端重新获取配置数据即可。<br>
##### Zk的使用场景二：负载均衡
`方式：`首先在Zookeeper上创建一个节点来进行域名配置，例如创建节点：/DDNS/app1，在这个节点上，每个应用都可以将自己的域名配置上去，如:192.168.0.1:8000<br>
在传统的DNS解析中，都不需要关心域名的解析过程，把这些工作都交给了操作系统的域名和Ip地址映射机制（本地HOST绑定）等，在DDNS中，域名的解析过程都是由一个应用自己负责的。`通常应用都会首先从域名节点中获取一份IP地址和端口的配置，进行解析。同时，每个应用还会在域名节点上注册一个数据变更Watcher监听，以便及时收到域名变更的通知`.<br>
##### Zk的使用场景三：命名服务
命名服务是分布式系统中比较常见的场景，在分布式系统中，被命名的实体通常可以是集群中的机器，提供的服务地址或远程对象等－－这些都可以统称它们为Name，其中较为常见的就是一些分布式服务框架（RPC，RMI）中的服务地址列表，通过使用命名服务，客户端应用能够根据指定名字来获取资源的实体，服务地址和提供者的信息等。<br>
广义上命名服务的资源定位都不是真正意义上的实体资源－－－在分布式环境中，上层应用仅仅需要一个全局唯一的名字，类似于数据库中的唯一主键。于是命名服务变成了在分布式全局唯一ID的分配。<br>
不使用UUID实现分布式全局唯一ID的原因：1.长度过长，2.含义不明<br>
利用zookeeper的特性：通过调用Zookeeper节点创建的API接口可以创建一个顺序节点，并且在API返回值中会返回这个节点的完整名字，利用这个特点就可以借助Zookeeper来生成全局唯一的ID了。
##### Zk的使用场景四：分布式协调／通知
分布式协调通知是分布式系统中不可缺少的一个环节，是将不同的分布式组件有机结合起来的关键所在。引入这样一个协调者，可以将分布式协调的职责从应用中分离出来，从而减少系统之间的耦合行，显著提高可扩展性。
##### Zk的使用场景五：集群管理

##### Zk的使用场景六：Master选举
##### ZK的使用场景七：分布式锁
###### 分布式拍他锁：
在分布式的环境下，创建分布式锁的过程可以如下：`获取锁:`在/exclusive_lock节点下创建临时节点/exclusive_lock/lock，由于Zk可以保证在所有的客户端中，最终只有一个客户端能够创建成功，那么就可以认为该客户端获取了锁。同时所有没有获取到锁的客户端就需要到/exclusive_lock节点上注册一个子节点变更的Watcher监听，以便实时监听到lock节点的变更情况。<br>
`释放锁:`/exclusive_lock/lock是一个临时节点，因此在以下两种情况下都有可能被释放：<br>
当钱获取所得客户端发生宕机，那么Zookeeper上的这个临时节点就会被移除。<br>
正常执行完逻辑后，客户端就会主动将自己创建的临时节点删除。<br>
无论哪种情况，Zk都会通知所有在/exclusive_lock节点上注册了自节点变更Watcher监听的客户端。这些客户端在获得通知后，再次重新发起分布式锁获取。<br>
###### 分布式共享锁:
`定义锁：`和拍他锁一样，同样是通过Zk上的数据节点来表示一个锁，是一个类似/shared_lock/host-R-00001的临时顺序节点，。<br>
`获取锁：`在需要获取共享锁时，所有客户端都会到/shared_lock这个节点创建一个临时顺序节点，如果当前是读请求，就创建/shared_lock/192.168.0.1-R-0001而如果是写请求就创建/shared_lock/192.168.0.1-W-0001的节点。<br>
`判断读写顺序：` 不同的事务都可以同时对一个数据对象进行读取操作，而更新操作必须在当前没有任何事物进行读写操作的情况下进行。Zookeeper是这样确定分布式读写顺序的：<br>
1.创建完节点后，获取/shared_lock节点下的所有子节点，并对该节点注册自节点变更的Watcher监听。<br>
2.确定自己的节点序号在所有子节点中的顺序。<br>
3.根据节点的创建顺序可以查看比自己序号小的节点是读还是写节点。来决定是否进行等待还是获得共享锁进行逻辑执行,如果自己不是序号最小的子节点，那么就需要进入等待。<br>
4.接收到Watcher通知后，重复1步骤。<br>
##### Zk的使用场景八：分布式队列
#### zk的简单使用:
zk选master：<br>
项目目录：https://github.com/asdgh1000/demo/tree/master/zookeeper-test
###1.7 分布式服务框架：Dubbo
Dubbo核心部分包括：
 `1.远程通信：`提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及"请求－响应"模式的信息交换方式。<br>
2.`集群容错：`提供基于接口方法的方法的远程过程调用，包括对多协议的支持，以及对软负载均衡，失败容错，地址路由和动态配置等集群特性的支持。<br>
3.`自动发现：`提供基于注册中心的目录服务，使服务消费方能动态地查找服务提供方，使地址透明，使服务提供方可以平滑的增加或减少机器。<br>
此外Dubbo框架还包括负责对象序列化的Serialize组件，网络传输组件Transport，协议层Protocol以及服务注册中心Registry等。<br>
其中注册中心是RPC框架最核心模块之一，用于服务的注册和订阅。因为Zk是一个树形结构的目录服务，支持变更推送，因此非常适合作为Dubbo的注册中心。<br>

#### RPC:
远程过程调用协议,它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。在分布式环境中，RPC保证了服务器节点之间的消息传输交互，让用户感觉是完全透明的。
#### Dubbo基本使用：
alibaba开源分布式服务框架，可以使用dubbo来处理分布式服务问题
随着网站规模的逐渐扩大分布式服务架构以及流动计算架构势在必行，此时，用于提高业务复用及整合的 分布式服务框架(RPC) 是关键。
#### Dubbo的简单实用：
利用zk作为数据发布订阅中心，实现服务的远程调用：<br>
项目目录：https://github.com/asdgh1000/demo/tree/master/dubbo-test
###1.8 分表分库方案
#### 1分表
#### 分表优势
对于大量数据存储在同一张表，会降低查询效率，此时分表能使查询效率加快很多。（随数据增长，查询速度不是线性下降，而是指数下降。）
##### 分表常见方案：
通常来说可以使用id作为分表字段，例如一张大表要分成200张表，可以使用（id%200）这样的方式确定是应该存储在张表。不过这样无论什么样的查询都需要加上id作为查询条件，否则会无法定位到相应的分表。
##### 分表瓶颈：
分表能解决单表数量过大带来的查询效率下降的问题，但是没办法给数据库的并发性能提供帮助，因为仅有Master节点能进行修改操作。面对高并发的读写操作，当数据库Master服务无法承载写操作压力时，不管如何扩展slave服务器，都没有帮助。

#### 2分库
#### 分库优势
对数据库拆分可以提高写的能力，从而提高系统的并发处理能力。
##### 分库常见方案
与分表策略类似，分库也可以采用通过一个关键字取模的方式来对数据访问进行路由。
例如分成200个库，则使用（id%200）的方式确定具体在哪个库.
##### 分库瓶颈 
单纯进行分库操作而不分表无法解决海量数据存储在一张表的问题。此时可以进行分表分库方案。
##### 数据库垂直拆分／水平拆分
###### 垂直拆分
把一个数据库中不同业务单元的数据分布到不同数据库中。(仍然有存在局部过热的情况)
####### 带来的问题
1.单机ACID保证被打破了，要么放弃原本了单机事务，要么引入分布式事务。<br>
2.一些join操作变的很困难，因为数据可能已经在两个数据库中了，所以不能很方便的利用数据库自身的join，需要其他方式解决。<br>
3.靠外键进行约束的场景会受影响。<br>
###### 水平拆分
把一个数据库中相同业务单元的数据分布到不同数据库中。(引入的问题会更加复杂)
####### 带来的问题
1.同样有可能有ACID被打破的情况。
2.同样可能有Join操作被影响的情况。
3.靠外键去进行约束场景会有影响。
4.依赖单库的自增序列生成唯一ID会受影响。
5.针对单个逻辑意义上的表的查询要跨库了。


#### 3分表分库
##### 分库分表优势
同时扩展系统的并发处理能力，又提升表的查询性能。
##### 分表分库常见方案
常见路由策略如下：<br>
中间变量＝id%(库数量*每个库的表数量)；<br>
库＝取整(中间变量/每个库的表数量)<br>
表＝中间变量%每个库的表数量<br>
#### 存在问题
1.原本表的事务上升成为分布式事务。<br>
2.由于纪录被切分到不同的库与不同的表中，难以进行多表关联查询
###1.9 分布式／集群session管理
#### 1 Session Sticky
##### 方案
利用类似源ip地址hash的方式使，每个相同客户端的请求总是访问一个相同的服务。
##### 瓶颈
1.如果任意一台web服务器宕机或重启，那这台机器上的会话就会丢失，如果会话中有登录状态数据，那么用户就要重新登录了。<br>
2.会话标识是应用层的信息，那么负载均衡器将要一个会话的请求都保存到同一个web服务器上的话，要进行应用层（第7层）的解析，这个开销比第四层大。<br>
3.负载均衡器变成了一个有状态的节点，要将会话保存到具体web服务器的映射。和无状态的节点相比，内存消耗会更大，容灾方面会更麻烦。<br>
#### 2 Session Replication
##### 方案
使用会话数据同步，通过同步保证不同web服务之间的session数据一致。
##### 瓶颈
1.同步session数据造成了网络带宽开销，只要session数据有变化，就需要将数据同步到所有机器，机器越多同步带来的网络开销就越大。<br>
2.没台web服务器都要保存session数据，如果整个集群的session数据很多，没太服务器保存的session内容占用会很多。<br>
#### 3 Session 数据集中存储
##### 方案
使用缓存或数据库集中存储起来，然后不同web服务器从同样的地方来获取session。
##### 瓶颈
1.读写session数据引入了网络操作，这相对于本机数据来说，问题就在于存在时延和不稳定性，不过我们的通讯基本都是发生在内网，问题不大。
2.如果集中存储Session的机器或集群有问题，就会影响我们的应用。
#### 4 Cookie Based
##### 方案
将Session数据放在Cookie中，然后在web服务器上从Cookie中生成对应的session数据。相比前面的集中存储，这个方案不会依赖外部的一个存储系统，也不存在从外部系统获取，写入Session网络延时。
##### 瓶颈
1.Cookie长度限制，会限制Session长度限制。<br>
2.安全性：Session数据本来都是服务端数据，而这个方案是让这些服务端数据到了外部网络及客户端，因此存在安全性问题。我们可以对写入Cookie的Session数据进行加密，不过对于安全来说，物理上不能接触才是安全的。<br>
3.消耗带宽，这里指的不是内部web服务器之间的带宽消耗，而是我们数据中心的整体外部带宽消耗。<br>
4.性能影响。每次HTTP请求和响应都带有Session数据，对web服务器来说，在同样处理情况下，响应的结果输出越少，支持的并发请求就会越多。<br>
### 1.10缓存
#### Redis
##### 扩展redis
### 1.11 数据库
#### 1.1 SQL语句执行过程详解
第一步：客户端将语句发送给数据库服务端<br>
在客户端执行比如select语句的时候，客户端先将SQL发送给服务端，在客户端连接服务端的同时，服务端也会对应的创建一条进程，但是这个进程的作用和客户端的进程不一样，客户端进程主要负责发送sql语句，而服务端的进程则负责sql的相关处理。<br><br>
第二步：语句解析<br>
语句解析不是一个动作，而是多个动作完成的：<br>
1.查询高速缓存（library cache）<br>
服务器进程在接到客户端传送过来的SQL语句时，不会直接查询数据库，而是先在数据库的高速缓存中去查找，是否存在相同的语句执行计划。（要注意的是这部分的缓存和一般意义上客户端软件的缓存不是一回事，这边的数据库缓存只要修改了数据库信息，立马会反应在数据库缓存之上）。<br>
2.语句合法性检查(data dict cache)<br>
当在高速缓存中找不到对应的sql语句时，则服务器进程就会开始检查这条语句的合法性。主要是对语法进行检查，在检查过程中不会涉及表名，列名，只是语法的检查。<br>
3.语言含义检查(data dict cache)<br>
对语句的字段进行检查，如表，字段是否在数据库中，所以,有时候我们写 select 语句的时候,若语法
与表名或者列名同时写错的话,则系统是先提示说语法错误,等到语法完全正确后,再提示说列名或表名
错误。<br>
4.获得对象解析锁(control structer)<br>
当语法、语义都正确后,系统就会对我们需要查询
的对象加锁。这主要是为了保障数据的一致性,防止我们在查询的过程中,其他用户对这个对象的结构发
生改变。<br>
5.数据访问权限的核对(data dict cache)<br>
要注意的是，在进行了语句合法性，语言含义检查和对象锁获取后才是访问权限的核对。<br>
6.确定最佳执行计划<br>
一般在应用软件开发的过程中,需要对数据库的 sql 语言进行优化,这个优化的作用要大大地大
于服务器进程的自我优化。所以,一般在应用软件开发的时候,数据库的优化是少不了的。当服务器进程
的优化器确定这条查询语句的最佳执行计划后,就会将这条 SQL 语句与执行计划保存到数据高速缓存
(library cache)。如此的话,等以后还有这个查询时,就会省略以上的语法、语义与权限检查的步骤,
而直接执行 SQL 语句,提高 SQL 语句处理效率。<br><br>
第三步:语句执行<br>
等到语句解析完成之后,数据库服务器进程才会真正的执行这条 SQL 语句。这个语句执行也分两
种情况。<br>
一是若被选择行所在的数据块已经被读取到数据缓冲区的话,则服务器进程会直接把这个数据传递
给客户端,而不是从数据库文件中去查询数据。
若数据不在缓冲区中,则服务器进程将从数据库文件中查询相关数据,并把这些数据放入到数据缓冲
区中(buffer cache)。<br><br>
第四步:提取数据<br>
当语句执行完成之后,查询到的数据还是在服务器进程中,还没有被传送到客户端的用户进程。所以,
在服务器端的进程中,有一个专门负责数据提取的一段代码。他的作用就是把查询到的数据结果返回给
用户端进程,从而完成整个查询动作。从这整个查询处理过程中,我们在数据库开发或者应用软件开发过
程中<br>

<br>注意：<br>
一是要了解数据库缓存跟应用软件缓存是两码事情。数据库缓存只有在数据库服务器端才存在,在
客户端是不存在的。只有如此,才能够保证数据库缓存中的内容跟数据库文件的内容一致。才能够根据
相关的规则,防止数据脏读、错读的发生。而应用软件所涉及的数据缓存,由于跟数据库缓存不是一码事
情,所以,应用软件的数据缓存虽然可以提高数据的查询效率,但是,却打破了数据一致性的要求,有时候
会发生脏读、错读等情况的发生。所以,有时候,在应用软件上有专门一个功能,用来在必要的时候清除
数据缓存。不过,这个数据缓存的清除,也只是清除本机上的数据缓存,或者说,只是清除这个应用程序
的数据缓存,而不会清除数据库的数据缓存。<br>

###10 中间件
#### 1 远程过程调用和对象访问中间件
主要解决分布式环境下应用的互相访问问题，这也是支撑应用服务化的基础
#### 2 消息中间件
解决应用之间的消息传递，解耦，异步等问题。
##### 消息发送一致性定义
消息发送一致性是指产生消息的业务动作与消息发送的一致，就是说如果业务操作成功了，那么由这个操作产生的消息一定要发送出去，否则就丢消息了。另一方面，如果这个业务行为没有发生或者失败，就不应该把消息发送出去。
###### 存在的一致性问题
无论是先发消息后执行业务逻辑，还是先执行业务逻辑后发消息，都会产生消息不一致性问题（消息系统可能挂掉）
##### JMS (Java Message Service)
###### JMS中的模型
JMS Queue 模型：<br>
PTP方式，一个消息只能被一个应用消费。<br>
JMS Topic 模式：<br>
Pub/Sub方式，所有监听了一个topic的应用都可以收到同一条消息。<br>
###### 概念
1.Destination:消息所走通道的定义，也就是用来定义消息从发送端发出后要走的通道，而不是最终接收方。Destination属于管理类的对象。<br>
2.ConnectionFactory:用于创建连接的对象。ConnectionFactory属于管理类的对象。<br>
3.Connection:连接接口,所负责的工作主要是创建Session。<br>
4.Session:会话接口，消息的发送者，接受者以及消息对象本身都由这个会话创建。<br>
5.MessageConsumer:消息的消费者，订阅消息并处理消息的对象。<br>
6.MessageProducer:消息的生产者，用来发送消息的都喜庆。<br>
7.XXXMessage:包括（BytesMessage,MapMessage,ObjectMessage,StreamMessage,TextMessage）五种。<br>
###### XA接口
####### 存在问题
1.使用XA接口能解决消息和业务逻辑发送一致性问题，但是同时也会引入分布式事务，这会带来开销并增加复杂性<br>
2.对于业务操作有限制，要求业务操作必须支持XA协议，才能够与发送消息一起来做分布式事务，这会成为一个限制，因为并不是所有需要与发送消息一起做成分布式事务的业务操作都支持XA协议。
ConnectionFactory,Connection,Session有着对应的XA接口，因为事务控制实在Session层面上的，而Session是通过Connection创建的，Connection是通过ConnectionFactory创建的，所以这三个接口需要有对应的XA接口。（Session，Connection，ConnectionFactory在Queue和Topic模型下对应的各个接口也存在XA系列接口）。


#### 3 数据访问中间件
主要解决应用访问数据库的共性问题的组件。





