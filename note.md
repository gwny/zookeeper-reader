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
```