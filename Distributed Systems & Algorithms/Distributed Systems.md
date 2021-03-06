# 一、分布式系统概述和介绍
## 1. 分布式和并行式的区别：
分布式的进程通信会有时延（因为是用消息通信的）；MPP是立刻传输的（shared memory）
分布式的时间戳很难比较，而MPP的时间戳基本就是那个时间。
总结：分布式no common time/state <=> MPP：common time/state

## 2. 分布式的问题和挑战：
No global time (concurrent issues);
No global state (partial failure/outdate message);
No determine behavior (hard to debug)

## 3. RPC过程：
Client 方面：compute->marshalling->send;
Transmission;
Server方面：receive->demarshalling->calculate->marshalling->send;
Transmission;
Client 方面：receive->demarshalling;
[RPC Process](https://github.com/luckyredpanda/luckyredpanda.github.io/blob/master/pictures/ds-RPC.png)
注意：在这里，发送可以1. 类似于单指令，一个时间段内只发送一个2. 流水线发送，C发送一个后持续发送，S在处理完上一个后接着处理下一个。

## 4. Marshalling:
process of flattening/serializing of objects and primitive data into presentation.（将对象和原始数据展平/序列化为表示的过程。）
有两种Marshalling的方法：
1. Receiver处理数据
2. S/R共同用一种标准，可以有更好的通用性。

# 二、分布式系统的问题
## 1. No global time:
在分布式系统中，有多少个系统就有多少个时钟。时钟经过协调以保持它们的一致性，但没有一个时钟具有准确的时间。即使时钟有些同步，每个组件上的各个时钟也可能以不同的速率或粒度运行，导致它们仅在一个本地时钟周期后才不同步。
而且，因为没有global time, 人们很难知道order of events->这会导致很多问题。所以我们必须process的关系（谁先谁后）

这里有一个先后关系的判断：（就拿Lamport‘s logical clock的图所示）
1. 同一个进程 c在d前所以c在d前发生。
2. 同一个信息d发送到e所以d在e前。
3. 综合1和2，c在f前。

### 1. NTP(但是应用中很难实现)
[NTP](https://github.com/luckyredpanda/luckyredpanda.github.io/blob/master/pictures/ds-delay%26offset.png)
就是通过发送消息，统计两个服务器的时钟然后用数学方法来保持时钟一致。

### 2. Lamport‘s logical clock
[Lamport‘s logical clock](https://github.com/luckyredpanda/luckyredpanda.github.io/blob/master/pictures/ds-Lamport%E2%80%98s%20logical%20clock.png)
就是给各个时间点加上逻辑数字，只有满足1，2，3的时候节点数字相加。也就是同步。
但是这个算法有问题就是b和e是平行的，但是数字不一样->有时无法通过数字反推顺序。

### 3. vector clock
[vector clock](https://github.com/luckyredpanda/luckyredpanda.github.io/blob/master/pictures/ds-vector%20clock.png)
向量时钟，就是加维度。这样这个问题就解决了。

## 2. No global state:
在分布式垃圾收集、分布式死锁检测、分布式终止检测、分布式调试中会有问题。
由于分布式系统中物理时间无法完全同步，因此无法在特定时间内收集系统的全局状态。->Cuts 来记录全局状态。
Snapshots: consistent cuts
Chandy-Lamport算法
[Chandy-Lamport](https://github.com/luckyredpanda/luckyredpanda.github.io/blob/master/pictures/Chandy-Lamport.png)
1. 某个进程在特定时间停止处理新数据然后dump内存状态并记录时间点。
2. 在该节点发出snapshot的请求，然后在该点以传输速率相同的斜率向其他进程发送snapshot请求。
3. 然后所有节点都处理marker前的数据，marker后的数据先缓存记录，等所有节点都处理完marker前的数据并且都导出自己的local snapshot后，这个global snapshot就完成了。

## 3. 一些合作问题
-failure detection：unreliable and reliable
-互斥：在分布式系统中，排他性的资源访问方式，叫做分布式互斥，而这种互斥访问的共享资源就叫做临界资源。可以让中心服务器分配、可以rings（击鼓传花）。
-选举：找出coordinator
-共识: Raft

# 三、云
## 1. MapReduce and GFS
【TK1✖️MIT6.824】读MapReduce论文 - Glonnaz009的文章 - 知乎
https://zhuanlan.zhihu.com/p/377463807

【TK1✖️MIT6.824】读Google File System论文 - Glonnaz009的文章 - 知乎
https://zhuanlan.zhihu.com/p/377703976

## 2. Paxos
每个参与进程扮演三个角色；提议者、接受者和学习者。
Paxos算法详解 - 祥光的文章 - 知乎
https://zhuanlan.zhihu.com/p/31780743

•  第一阶段：Prepare阶段。Proposer向Acceptors发出Prepare请求，Acceptors针对收到的Prepare请求进行Promise承诺。
•  第二阶段：Accept阶段。Proposer收到多数Acceptors承诺的Promise后，向Acceptors发出Propose请求，Acceptors针对收到的Propose请求进行Accept处理。
•  第三阶段：Learn阶段。Proposer在收到多数Acceptors的Accept之后，标志着本次Accept成功，决议形成，将形成的决议发送给所有Learners。

## 3. Raft
Raft算法详解 - 祥光的文章 - 知乎
https://zhuanlan.zhihu.com/p/32052223


总结：http://www.cyc2018.xyz/%E5%85%B6%E5%AE%83/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/%E5%88%86%E5%B8%83%E5%BC%8F.html#%E5%8D%95%E4%B8%AA-candidate-%E7%9A%84%E7%AB%9E%E9%80%89
