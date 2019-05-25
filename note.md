# 服务端

服务端的启动：

```
bin/zkServer.sh start
```

入口就看bin/zkServer.sh

```
其中，. "$ZOOBINDIR/zkEnv.sh"执行语句设置一些环境变量。
ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain"设置了启动的入口。
```

Java代码：

```
server的主入口：org.apache.zookeeper.server.quorum.QuorumPeerMain

// 监听的端口号
	protected InetSocketAddress clientPortAddress;
	// 内存数据库快照的存储位置
	protected String dataDir;
	// 日志存储位置
	protected String dataLogDir;
	// 每一个时钟周期的毫秒数,zookeeper的基本时间单位
	protected int tickTime = ZooKeeperServer.DEFAULT_TICK_TIME;
	
Running Replicated ZooKeeper
在单机模式下运行zookeeper进行开发测试非常方便。但是在生产环境中，应该以集群方式运行。运行相同程序的同一组服务器称作quorum，而且quorum中所有的服务器有着相同的配置文件。配置和单机模式的配置类似，但是有一些小的区别，举个例子：
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
  其中，initLimit是quorum中的服务器连接leader的超时时间，syncLimit是quorum中的服务器连接leader的过期时间。配置的值都是多少个tickTime，例如initLimit配置的是5，tickTime配置的是2000毫秒，则initLimit的超时时间为10秒。
  server.X列表配置了组成Zookeeper服务的服务器地址，当服务器启动时，可以通过在数据目录中查找文件myid来知道它是哪个服务器，该文件包含服务器编号。
  从server.X的配置中可以看到，有两个端口号：2888和3888。quorum中的服务器通过前一个端口号(2888)连接quorum中的其他服务器，服务器之间的通信是必要的，例如，就更新的顺序达成一致。更具体地说，Zookeeper服务使用这个端口号连接quorum中的followers和leader。当一个新的leader产生时，follower使用此端口开启一个和leader之间的TCP连接。后一个端口号(3888)用于leader选举。
  
节点有三种状态：
leader：负责维护节点信息。
follower：负责接收转发client的请求。
observer：只是监听leader的变化。

节点配置类型（LearnerType），默认为PARTICIPANT：
PARTICIPANT：可以投票和参加leader选举，可以成为leader或follower。
OBSERVER：可以投票，但是不能参加leader选举，可以成为observer。

PurgeTxnLog：
这个类在zookeeper的服务器上定时运行，用来清理dataDir下的快照文件和dataLogDir目录下的文件，保留最后n个快照文件和对应的日志。
dataDir下文件命名规则: snapshot.zxid
dataLogDir下文件命名规则： log.zxid

ZookeerServerMain：
ZooKeeperServerMain.main(args)，启动单机模式

java并发类CountDownLatch：
一个用于同步的辅助类，基于JDK中AbstractQueuedSynchronizer类实现，用于一个或多个线程等待其他线程的操作完成。
原理：初始化设置一个数值count，调用await()方法阻塞线程，调用countDown()方法，count-1，count为0时继续其他操作。

ServerCnxnFactory:
管理服务端和客户端的连接。有两种实现，分别为NIOServerCnxnFactory和NettyServerCnxnFactory，通过读取系统配置zookeeper.serverCnxnFactory反射实例化对应的类。默认创建NIOServerCnxnFactory.

NIOServerCnxn：
使用NIO处理服务端和客户端的通信，每个客户端有一个，但只有一个线程在进行通信。主要是doIO()方法。

org.apache.zookeeper.server.ZooKeeperServer：
实现一个单机版的ZookeeperServer。

Java NIO:
IO和NIO对比：
1. IO面向流，NIO面向缓冲区
2. IO是阻塞的，NIO是不阻塞的
3. NIO有选择器，IO没有
NIO三个概念：Channle、Buffer、Selector
传统阻塞式I/O问题：
1. 建立连接后，当前线程被挂起，直到有数据达到才唤醒，否则一直占着内存资源，如果需要读取其他连接的数据，必须启动另外的线程。连接数过多的时候，内存资源会被大量消耗。而且大量的线程切换也浪费系统资源。

Channle：
代表一个可以执行I/O操作的实体，例如硬盘设备、文件，网络套接字等。Channle可以读取数据到缓冲区，也可以将缓冲区的数据写到Channle。
Buffer:
capacity：容量    position：指针当前位置     limit：读写边界
Selector：
将Channle注册到Selector，Selector调用select()方法等待对应的事件发生（Accept，Connect，Read ，Write）。


```