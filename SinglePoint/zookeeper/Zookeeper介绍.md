# Zookeeper介绍
参考：https://zookeeper.apache.org/doc/trunk/zookeeperOver.html
## 1   ZooKeeper: 针对分布式应用程序的协调服务者
ZooKeeper是一个面向分布式系统的分布式，开源的协调服务者。它提供了一组简单的原语用于分布式应用程序能够基于此实现同步，配置管理，分组，命名等更高层次的服务。在设计上做到了易于编程，使用文件系统的目录树结构的数据模型。基于Java运行但同时支持Java & C。
协同服务要正确实现是相当难的。它很容易遇到如：条件冲突或者死锁等错误。ZooKeeper实现的目标是降低分布式系统从头开始实现协调服务的复杂度。
### 1.1 整体设计 

**ZooKeeper比较简单。** ZooKeeper允许分布式进程通过一个基于标准文件系统下易于管理的，共享的，分级的命名空间来相互通讯。命名空间由在ZooKeeper中称之为“znode”的数据寄存器组成。这些寄存器类似文件和目录。和典型的文件系统有所差异的是，标准的文件系统是基于存储，ZooKeeper将数据在内存中保持，这意味着ZooKeeper可以实现高吞吐与低延迟。
ZooKeeper在高性能，高可用，有序访问上做了大量的工作。在性能方面，ZooKeeper可以被用在大型分布式系统中。在可靠性方面，它不会有单点失败的问题。在有序访问方面，可以将复杂的同步原语在客户端实现。

**ZooKeeper比较复杂。** 类似协同的分布式进程一样，ZooKeeper通过一个集群主机实现自我复制。
![zkservice](../../static/zkservice.jpg)

构成ZooKeeper服务的server必须相互感知。它们在内存中维护状态，持久存储事务日志与快照。只要大部分servers节点有效，ZooKeeper服务就是可用的。
Client连接单个ZooKeeper server。client维护TCP连接，通过连接发送请求，获取响应，获取监听的事件，发送心跳等。如果连接到server的TCP连接断开了，客户端会连接另一个不同的server。

**ZooKeeper是有序的。** ZooKeeper为每一次更新确定一个数字用以确定所有ZooKeeper事务的顺序。后续操作可以使用这种顺序来实现更高级别的抽象，例如同步原语。

**ZooKeeper很快。** 在读主导的场景中特别快。ZooKeeper应用运行在数千台机器上时，在读写比大约10：1的情况下，可以得到最佳性能。

### 1.2 数据模型与分级命名空间
ZooKeeper提供的命名空间近似标准文件系统。一个名字就是一个由(/)分隔的路径元素序列。每个ZooKeeper中的node的命名空间由路径来确定。
![zknamespace](../../static/zknamespace.jpg)

### 1.3 节点与临时节点
与标准文件系统不同，每个ZooKeeper命名空间的的节点都可以持有相关的数据以及子节点。这有点像文件系统允许一个文件同时也是目录。(ZooKeeper设计于存储协同数据：状态信息，配置，位置信息等等。所以每个节点通常存储的信息很少，在数字节到数千字节区间内。)我们使用znode术语来表明正在谈论的ZooKeeper数据节点。
znode维护一个统计结构，其中包括数据变更，ACL更改和时间戳的版本号。其允许缓存验证和协同更新。每当znode数据发生变更时，版本号就会增加。例如，每当客户端请求数据时，同时也会拿到数据的版本。
存储在命名空间每个znode的数据是原子方式读写。读操作获取znode相关联的所有数据字节，写操作覆盖所有数据。每个node都有一个权限控制列表(ACL)控制谁可以做什么。
ZooKeeper也有临时节点的概念。只要创建znode的会话存活，那这些znodes就会存在。当会话结束，znode被删除。当你要做某些实现时，临时节点很有用。

### 1.4 条件更新与订阅
ZooKeeper支持订阅的概念。客户端可以在znode上设置订阅。当znode发生变更时，一个订阅会被触发与移除。当一个订阅被触发，客户端会收到一个znode变更的通知消息。如果客户端与ZooKeeper servers之间的链接中断，客户端会收到一个本地通知。这些是订阅的一些使用场景。

### 1.5 承诺
ZooKeeper既快又简单。其目标是为组建更复杂的服务(如同步)提供一套可靠的基础。包括：
*   顺序一致性 - 从客户端发送来的更新请求将按照其发送顺序来应用。
*   原子 - 更新只有成功或失败，没有其它结果。
*   统一系统镜像 - 客户端无论连接到哪个server，都会看到同样的内容。
*   可靠性 - 当一个更新被应用，它将从那个时刻开始持续到客户端覆盖更新完成。
*   时间线性 - 客户端的系统视图在一定时间内确保是最新的。

### 1.6 简单API
ZooKeeper的一个设计目标是提供一个简单编程接口。结果是，仅支持如下操作：
* create 在tree的某个位置上创建一个node
* delete 删除node
* exists 测试node是否在指定位置存在
* get data 从node读数据
* set data 将数据写入node
* get children 检索一个node的子节点集合
* sync 等待数据被传播

### 1.7 实现
下图展示了ZooKeeper服务的高级组件。除了请求处理器外，组成ZooKeeper服务的每个服务器都会复制所有组件的副本。
![zkcomponents](../../static/zkcomponents.jpg)

复制的数据库是包含整个数据树的内存数据库。更新操作会记录日志到磁盘以获取可恢复性，写入先序列化到磁盘后再应用到内存数据库。
每个ZooKeeper server服务多个client。这些client链接到一个server来提交irequests。从每个server数据库的本地副本来处理读请求。通过约定协议请求可以更改服务状态，写入请求等。约定协议的一部分是，所有的来自客户端的写入请求都会转发到一个单一server，其称之为leader。其余的ZooKeeper服务器称之为 follower(跟随者)，接收来自leader的消息推送并将之传递。消息传递层负责在失败时替换leaders，并保持leaders与followers的同步。
ZooKeeper使用自定义的原子消息协议。因为消息传递层是原子的，所以ZooKeeper可以保证本地副本不会发散。当leader收到写请求，其会计算写入系统时的状态，并将其转换为捕获新状态的事务。

### 1.8 使用
ZooKeeper的编程接口设计的很简单。基于此你可以实现更高阶的操作，如同步原语，组成员资格，所有权资格等。

### 1.9 性能
ZooKeeper的设计提供了高性能。ZooKeeper在雅虎的开发团队的研究结果表明它的确如此。
在读远大于写的应用场景中，可以提供很高的性能。因为写设计同步状态到所有的servers。(协调服务通常读远大于写。)
![zkperfRW](../../static/zkperfRW-3.2.jpg)

ZooKeeper吞吐量基于读写比率变化图形，是ZooKeeper 3.2版本运行在至强双核2Ghz和双SATA 15K RPM驱动器的服务器上运行的吞吐量图。一个驱动器被专用做ZooKeeper日志设备。快照写入系统驱动器。写入请求是1k写入，读请求也是1k读取。“Servers”表明ZooKeeper集群的大小，构成统一的服务。大约使用其它30台机器来模拟client。ZooKeeper集群被设置为leaders不允许接受来自client的链接。

在3.2版本中，R/W性能比之前的3.1版本提高了约2倍。

基准测试也表明其可靠。在可靠性方面，存在错误显示部署如何响应各种故障。图中标有的异常事件有：
1. follower的失败和恢复
2. 不同follower的失败和恢复
3. leader的失败
4. 两个follower的失败和恢复
5. 另一个leader的失败

### 1.10 可靠性
为了观察随着时间的推移失败的注入中系统的表现，我们在7台机器上运行ZooKeeper服务。我们执行和之前一样的饱和基准，但是这次我们将写入百分比控制在30%，这是我们预期工作量的保守比例。
![zkperfreliability](../../static/zkperfreliability.jpg)

从图中可以得到一些重要的观察结果。首先，如果followers失败且恢复的非常迅速，ZooKeeper可以在失败的情况下保持高吞吐量。但也许更重要的是，leader选举算法可以使得系统恢复的足够快速从而防止吞吐量大幅下降。在我们的观察中，ZooKeeper需要不到200ms的时间来选举一个新的leader。第三，当followers恢复后，ZooKeeper一旦开始处理请求，就能够再次提高吞吐量。

### 1.11 ZooKeeper项目
ZooKeeper已经被成功的在众多工业项目中使用。它在雅虎被用作协调与失败恢复服务Message Broker，这是一个高度可扩展的发布订单月系统，管理者数千个用于复制和数据传送的主题。还被用于雅虎爬虫的爬取服务，同样也管理失败恢复等处理。一些雅虎广告系统也在使用ZooKeeper实现可靠服务。