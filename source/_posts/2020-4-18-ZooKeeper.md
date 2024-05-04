---
layout: post
title: ZooKeeper算法
tag: 技术笔记
date: 2020-4-18
category: Technology blog
---
#### ZooKeeper如何保证数据一致性

#### 什么是ZooKeeper

它是一个分布式服务框架，是Apache Hadoop的一个子项目，主要是用来解决分布式应用经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等

在分布式场景中，ZooKeeper的应用非常广范，比如数据发布和订阅、命名服务、配置中心、注册中心、分布式锁等。举个例子：假设我们的程序是分布式部署在多台服务器上的，如果我们需要改变程序的配置文件，需要逐台机器去修改，非常的麻烦，现在把这些配置全部放到ZooKeeper上去，并且对相关应用程序进行监听，一旦配置信息发生变化，每个应用程序就会收到ZooKeeper的通知，然后从ZooKeeper获取新的配置信息应用到系统中

ZooKeeper提供了一个类似与Linux文件系统的数据模型，和基于Watcher机制的分布式事件通知，这些特性都依赖ZooKeeper的高容错数据一致性协议

那么问题来了，在分布式场景下，ZooKeeper是如何实现数据一致性的呢？

#### ZooKeeper目录结构

**bin**：防止运行脚本个工具脚本，如果是Linux环境还会有ZooKeeper的运行日志

**conf**：ZooKeeper默认读取配置的目录，里面会有默认的配置文件

**contrib**：ZooKeeper的拓展功能

**dist-maven**：ZooKeeper的maven打包目录

**docs**：ZooKeeper相关文档

**lib**：ZooKeeper核心的jar

**recipes**：ZooKeeper分布式相关的jar包

**src**：ZooKeeper源码

#### Zab一致性协议

ZooKeeper使用过Zab协议来保证分布式事务的最终一致性。Zab全称为ZooKeeper Atomic Broadcast，ZooKeeper 原子广播协议，支持崩溃恢复，基于该协议，ZooKeeper实现一种主备模式的系统架构来保持集群中的各个副本之间的数据一致性

其系统架构可以参考下图：

![Ciqah16O5QyAB4rJAAEiJ-4T3bE046](Ciqah16O5QyAB4rJAAEiJ-4T3bE046.png)

在ZooKeeper集群中，所有客户端的请求都是写入到Leader进程中的，然后，由Leader同步到其他节点，成为Follower。在集群数据同步的过程中，如果出现Follower节点崩溃或者Leader进程崩溃时，都会通过Zab协议来保证数据一致性。

Zab协议具体实现可以分为一下两部分

**消息广播阶段**

Leader节点接收事务提交，并且将新的Proposal请求广播给Follower节点，手机各个节点的反馈，决定是否进行Commit，在这个过程中，也会使用Quroum选举机制。

**崩溃恢复阶段**

如果在同步过程中Leader节点宕机，会进入崩溃恢复阶段，重新进行Leader选举，崩溃恢复阶段还包含数据同步操作，同步集群中最新的数据，保持集群的数据一致性。

整个ZooKeeper集群的一致性保证就是在上面两个状态之前切换，当Leader服务正常时，就是正常的广播模式；当Leader不可用时，则进入崩溃恢复模式，崩溃恢复阶段会进行数据同步，完成以后，重新进入消息广播阶段。

Zab协议中的Zxid

Zxid在ZooKeeper的一致性流程中非常重要，再详细分析Zab协议之前，先来看下Zxid的概念，Zxid是Zab协议的一个事务编号，Zxid是一个64位的数字，其中低32位是一个简单的单调递增计数器，针对客户端每一个事务请求，计数器加 1；而高 32 位则代表 Leader 周期年代的编号。

这里Leader周期的英文是epoch，可以理解为当前集群所处的年代或者周期，对比另外一个一致性算法Raft中的Term概念。在Raft中，每一个任期的开始都是一次选举，Raft算保证在给定的一个任期最多只有一个领导人

![Cgq2xl6O5QyAeZqMAAB5C-BWbeI425](Cgq2xl6O5QyAeZqMAAB5C-BWbeI425.png)

Zab协议的实现也类似，每当有一个新的Leader选举出现时，就会从这个Leader服务器上取出其本地日志中最大事务的Zxid，并从中读取epoch值，然后加1，以此作为新的周期ID。总结一下，高 32 位代表了每代 Leader 的唯一性，低 32 位则代表了每代 Leader 中事务的唯一性。

##### Zab流程分析

Zab的具体流程可以差分为消息广播、崩溃恢复和数据同步三个过程

![Ciqah16O5QyADv0LAAA84x9hlQc078](Ciqah16O5QyADv0LAAA84x9hlQc078.png)

**消息广播**

在ZooKeeper中所有的食物请求都有Leader节点来处理，其他服务器o为Follower，Leader将客户端的事务请求转换为事务Proposal，并且将Proposal分发给集群中其他所有的Follower。

完成广播之后，Leader等待Follower反馈，当有过半数的Follower反馈信息后，Leader将再次向Follower广播Commit信息，Commit信息就是将之前的Proposal提交。

这里的Commit可以对比SQL中的COMMIT操作来理解，MySQL默认操作模式是auto commit自动提交模式，如果你显式的开始一个事务，在每次变更之后都要通过COMMIT语句来确认，将更改提交到数据库中。

Leader节点的写入也是一个两步操作，第一步是广播事务操作，第二步是广播提交操作，其中过半数指的是反馈的节点数>=N/2+1，N是全部的Follower节点数量

消息广播的过程描述可以参考下图：

![Cgq2xl6O5Q2ASjMpAAHdAdF67vE736](Cgq2xl6O5Q2ASjMpAAHdAdF67vE736.png)

客户端的写请求进来之后，Leader会将写请求包装成Proposal事务,并添加一个递增事务ID，也就是Zxid，Zxid是单调递增的，以保证每个消息的先后顺序；

广播这个Proposal事务，Leader节点和Follower节点是解耦的，通信都会经过一个先进先出的消息队列，Leader会为每一个Follower服务器分配一个单独的FIFO队列，然后把Proposal放到队列中；

Follower节点收到对应的Proposal之后会把它持久到磁盘上，当完全写入之后发一个ACK给Leader；

当Leader收到超过半数Follower机器的ack之后，会提交本地机器上的事务，同时开始广播commit，Follower收到commit，完成各自的事务提交

**崩溃恢复**

消息广播通过Quroum机制，解决了Follower节点宕机的问题，但是如果在广播过程中Leader节点崩溃呢？

这就需要Zab协议支持的崩溃恢复，崩溃恢复可以保证Leader进程崩溃的时候可以重新选出Leader，并且保证数据的一致性。

崩溃恢复和集群启动时的选举过程是一致的，也就是说，下面几种情况都会进入崩溃恢复阶段：

> 初始化集群，刚刚启动的时候
>
> Leader崩溃，因为故障宕机
>
> Leader失去了半数的机器支持，与集群中超过一般的Follower失联

崩溃恢复模式将会重新开启新一轮选举，选举产生的Leader会与过半的Follower进行同步，使数据一直，当与过半的机器同步完成之后，就退出恢复模式，然后进入消息广播模式。

Zab中的节点有三种状态，伴随着的Zab不同阶段的转换，节点状态也在变化：

![Cgq2xl6O5-SASu1cAABBzFJh3s0114](Cgq2xl6O5-SASu1cAABBzFJh3s0114.png)

假设正在运行的集群有5台Follower服务器，编号分别是Server1，Server2，Server3，Server4，Server5，当前的Leader是Server2，若某一时刻Leader挂了，此时便开始Leader选举。

选举过程如下：

**1.各个节点变更状态，变更为Looking**

ZooKeeper中除了Leader和Follower，还有Observer节点，Observer不参与选举，Leader挂后，余下的Follower节点都会将自己的状态变更为Looking，然后开始进入Leader选举过程

**2.各个Server节点都会发出一个投票，参与选举**

在第一次投票中，所有的Server都会投自己，然后将各自投票发送给集群中的所有机器，在运行期间，每个服务器上的Zxid大概率不同

**3.集群接受来自各个服务器的投票，开始处理投票和选举**

处理投票的过程就是对比Zxid的过程，假定Server3的Zxid最大，Server1判断Server3可以成为Leader，那么Server1就投票给Server3，判断的依据如下：

首先选举epoch最大的

如果epoch相等，择选Zxid最大的

如果epoch和Zxid都相等，则选择server id最大的，就是配置zoo.cfg中的myid

在选举过程中，如果有节点获得超过半数的投票，则会成为Leader节点，反之则重新投票选举。

![Ciqah16O5Q2AL03bAADZjhkw3ys031](Ciqah16O5Q2AL03bAADZjhkw3ys031.png)

**4.选举成功，改变服务器的状态，参考上面这张图的状态变更**

数据同步

崩溃恢复完成选举之后，接下来的工作就是数据同步，在选举过程中，通过投票已经确认Leader服务器是最大Zxid的节点，同步阶段就是利用Leader前一阶段获得的最新Proposal历史，同步集群中所有的副本。

上面分析了Zab协议的具体流程，接下来我们对比一下Zab协议和Paxos算法

##### Zab和Paxos算法的联系与区别

Paxos思想在很多分布式组件中都可以看到，Zab协议可以认为是基于Paxos实现的

> 两者都存在一个Leader角色，负责去协调多个Follower进程的运行
>
> 都应用 Quroum 机制，Leader 进程都会等待超过半数的 Follower 做出正确的反馈后，才会将一个提案进行提交
>
> 在 Zab 协议中，Zxid 中通过 epoch 来代表当前 Leader 周期，在 Paxos 算法中，同样存在这样一个标识，叫做 Ballot Number
>

两者之间的区别是，Paxos是理论，Zab是实践，Paxos是论文性质的，目的是设计一种通用的分布式一致性算法，而Zab协议应用在ZooKeeper中，是一个特别设计的崩溃可恢复的原子消息广播算法。

Zab 协议增加了崩溃恢复的功能，当 Leader 服务器不可用，或者已经半数以上节点失去联系时，ZooKeeper 会进入恢复模式选举新的 Leader 服务器，使集群达到一个一致的状态。
#### 参考

> 《分布式技术原理与实战45讲》- 蒋越 
>