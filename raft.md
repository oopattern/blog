---
title: 2019-12-26 raft
tags: raft,distributed
renderNumberedHeading: true
grammar_cjkRuby: true
---

**问题**
1、同样是raft协议，为什么etcd只能支持10k的并发写入（几GB的存储），但是rocksdb却支持10k的写入，TG的存储？
2、etcd数据没有分片（每台server都是数据的全集），rocksdb数据有分片（相当于raft+leveldb）？
3、因此etcd只适用于配置（少量数据的更新），rocksdb适用于当做存储。

**raft协议**
中英文版论文：https://github.com/maemual/raft-zh_cn

**raft博客**
置顶：https://www.codedump.info/post/20180921-raft/
https://zhuanlan.zhihu.com/p/64405742

**raft概念**
1、leader选举：随机计时器解决冲突问题，错误每个服务器的选举时间。
2、日志复制：从leader复制给follower，保证follower和leader保持一致的状态。
3、安全性：在非拜占庭错误情况下，包括网络延迟、分区、丢包、冗余和乱序等错误都可以保证返回正确的结果给客户端。
3.1、可用性：集群中大多数机器可运行。
3.2、不依赖时钟：物理时钟错误或者极端的消息延迟才影响可用性问题。
4、复制状态机：每个服务器每个日志都按照相同的顺序执行相同的命令，一致性算法保证复制日志相同。
5、成员关系：
6、问题分解：raft算法分解为leader选举、日志复制、安全性、member ship等模块。
7、减少状态：服务器3个状态：leader、follower、candidate，复制日志连续不允许空洞，保证全局revision单调递增。

**复制状态机**
![enter description here](./images/1578449060551.png)

**服务器状态机标识**
![enter description here](./images/1578465872607.png)

**任期选举状态**
![enter description here](./images/1578468186724.png)

**server的三种状态**
![enter description here](./images/1578467097940.png)

**leader日志复制**
![enter description here](./images/1578485488233.png)

**安全性**
安全性保证每个server按照相同的顺序执行相同的命令。
![enter description here](./images/1578487367250.png)
问题1：候选者可能会被拒绝选举？候选者的Term比follower的Term小或者Term相同，但是最后日志索引小。
答：比较的是日志消息中的已提交消息的term和已提交消息的commitIndex。          
问题2：leader未提交的index可能被相互覆盖？
答：可能存在。
问题3：出现网络分区，一个term很大但是index很小的server是否可能获取leader，导致一致性被破坏？
答：不会，比较的是最后一条提交消息的term和commitIndex，而不是比较节点的term。
![enter description here](./images/1578808159571.png)
问题4：当leader从新选举成功时，为什么日志不能在当前任期提交，必须要等到下条日志一起确认提交？
答：如果重新选举成功的leader直接提交当前任期的日志，可能会出现该leader还没有给follower发送确认提交的日志，这时候其他server产生新的leader，然后新的leader可以覆盖原来leader提交的日志，这违反了raft规定同一个索引位置只能提交一次的原则。
问题5：如果leader没有重新选举，而是一直是leader，则该leader可以直接在当前任期提交日志，不需要在下条日志提交？
答：参考问题4的答案，这种情况下，就算leader挂掉，后面的server选举成功也会保留上个leader最近提交的日志，不会出现覆盖的情况。


**日志压缩**
![enter description here](./images/1578490911591.png)

**etcd应用**


**rocksdb应用**

