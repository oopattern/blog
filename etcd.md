---
title: 2019-12-26 etcd
tags: etcd,go
renderNumberedHeading: true
grammar_cjkRuby: true
---


**etcd安装运行**
单机运行：解压完直接执行 # nohup /path/to/etcd &
集群运行：参考文档：https://etcd.io/docs/v3.4.0/op-guide/clustering/ 模仿伪集群搭建。
``` shell
set +x

NAME1=infra0
NAME2=infra1
NAME3=infra2
HOST1=http://127.0.0.1:2379
HOST2=http://127.0.0.1:2479
HOST3=http://127.0.0.1:2579
PEER1=http://127.0.0.1:2380
PEER2=http://127.0.0.1:2480
PEER3=http://127.0.0.1:2580

# nohup ./etcd &

function StartService() {
  nohup ./etcd --name $1 --initial-advertise-peer-urls $2 \
  --listen-peer-urls $2 \
  --listen-client-urls $3 \
  --advertise-client-urls $3 \
  --initial-cluster-token etcd-cluster-sakulali \
  --initial-cluster infra0=$PEER1,infra1=$PEER2,infra2=$PEER3 \
  --initial-cluster-state new &
}

StartService $NAME1 $PEER1 $HOST1
StartService $NAME2 $PEER2 $HOST2
StartService $NAME3 $PEER3 $HOST3

echo "hello etcd..."
```

**etcd写入消息**
![enter description here](./images/1578978208738.png)
1、开始，客户端从leader或者从follower写入消息。如果从follower写消息会通过MsgProp转发给leader。
2、leader广播同步消息通知follower，消息包含数据内容Entry。
3、follower回复leader，leader统计回复过半数，进行数据提交操作，写入WAL、持久化（数据库）、和更新状态机，执行写操作。
4、leader广播提交消息通知follower，消息不包含数据内容，消息仅供follower提交。follower提交也会写WAL、持久化（数据库）和更新状态机。最终follower和leader保持同样的数据存储。
**问题1：** 为什么leader同步消息内容和同步提交索引给follower需要分开2步进行？如果由于leader崩溃导致无法发送同步提交索引给follower，下一个leader选举成功会包含上一个leader已经提交成功的内容吗？
答：
**问题2：**  follower什么时候将leader发送的同步消息保存写入WAL和持久化存储中？是收到leader的MsgApp（同步数据内容）后还是在收到leader的MsgApp（同步提交索引）后？
答：


**etcd写消息流程模块**
![enter description here](./images/1579224831663.png)
第一步：leader发送MsgApp消息给follower，包含消息内容
1.1 客户端发送消息给leader，如果是follower，会通过MsgProp消息转发给leader。
1.2、1.3 leader收到消息后将消息写入WAL日志中，用于故障恢复。
1.4、1.5 leader通知raft模块，将日志写入unstable（未持久化的存储）
1.6 leader通过MsgApp消息推送给follower，follower会将日志写入WAL中。
第二步：follower回复MsgAppResp消息给leader，表示接受leader的消息请求。
2.1 follower回复MsgAppResp响应通知leader。
2.2、2.3 通知etcd server，通过channel的方式。
2.4 leader将kv数据写入BoltDB持久化存储中。
2.5 leader回复客户端数据已经成功写入。
2.6 leader将日志写入storage中，表示日志数据已成功写入。
第三步：leader发送MsgApp消息给follower，不包含消息内容，包含提交内容。
3.1 leader发送MspApp给follower，不包含消息内容，仅包含提交内容。
3.2 follower收到MspApp，将消息写入持久化存储BoltDB中（应用到状态机）。
3.3 follower将数据写入storage中，storage是基于内存的数据，比较server已经进行持久化的数据。
第四步：follower回复MsgAppResp，表示接受leader的消息提交请求。
4.1 follower回复消息MsgAppResp给leader，表示客户端已经成功提交日志（已经进行持久化）。
4.2 leader收到follower过半回复时，回复客户端消息已经成功写入。
**测试1：**  leader发送MsgApp的消息内容给follower，设置follower不回复MsgAppResp，检查leader是否会将日志写入WAL中？还是说leader需要收到过半follower的回复后才会写入WAL？
**测试2：** leader什么时候才会写入持久化存储？是收到过半follower的MsgAppResp（消息内容），还是收到过半follower的MsgAppResp（提交内容），leader才会写入持久化存储？
**测试3：** follower什么时候写入WAL？什么时候写入持久化存储？ 
答：follower在收到leader的MsgApp（消息内容）就会写入WAL，在收到leader的MsgApp（提交内容）就会写入持久化存储（应用状态机）。


**WAL文件存储**


**KV数据存储**


**etcd快照**
1、快照的目的：raft不处理网络传输和持久化存储，通过接口的方式，由调用者自己实现。
2、快照的实现：Ready中满足snapshot条件，会触发snapshot操作，将数据写入持久化存储中。
3、快照恢复耗时：
4、快照和raft的关系：


**etcd客户端的使用**
1、kv数据每次修改对应一个全局的版本号，可以根据全局版本号回溯到某个具体版本的kv，可以用作配置历史回滚。
``` shell
# etcdctl get --prefix --rev=3 foo
```
2、应用进程断开连接或者etcd失败，下次应用进程重新连接etcd时，etcd可以把这段时间某个key（配置）的变化同步通知到应用进程。watch命令可以指定特定的版本进行观察。
``` shell
# etcdctl watch --rev=3 foo
```
3、对同一个key设置相同的val，全局版本号revison也会更新。
4、通过查看key值可以查看当前的全局版本号。
``` shell
# etcdctl get foo -w=json
```
5、etcd没有数据分片的功能，所有法定人数的机器保存一份同样的数据，而且数据量一般最多几GB，一般用于存配置信息或者机器ip，用作服务发现和负载均衡，不能当作分布式数据库（TiDB）来使用。
6、etcd会保留版本号，这样可以回溯到指定的版本号，但是版本号是全局的，对应该版本号所有的kv数据，需要定时compact压实版本号，否则版本号会堆积越来越多，导致磁盘和内存增加。

**raftexample使用**
``` shell
// 启动raft节点
# ./raftexample --id 1 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 12380 --join true
# ./raftexample --id 2 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 22380 --join true
# ./raftexample --id 3 --cluster http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 --port 32380 --join true
// 在终端写入kv
# curl -X PUT -d name=sakulali localhost:12380
// 在终端读出kv
# curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET localhost:12380
```

**etcd测试**
1、验证etcd客户端是否存在连接池？

2、etcd如何实现负载均衡？balance如何使用？

**etcd博客理解**
置顶：https://www.codedump.info/post/20180922-etcd-raft/
https://csunny.gitbook.io/etcd/arch
https://reading.developerlearning.cn/
http://lessisbetter.site/2019/09/05/etcd-raft-sources-structs/
https://reading.developerlearning.cn/reading/32-2019-03-02-etcd-raft/
https://blog.betacat.io/

**etcd模块关系**

**etcd功能的实现**
1、gRPC通信的实现。

2、客户端通信和raft组件交互的过程，包括leader选举和写操作的请求过程。

3、数据存储格式定义，索引和版本号的定义。

4、如何开启日志？

**etcd数据结构**
1、node和raft的关系，node通过通道完成和raft消息的通信。
2、版本和快照如何同步？
3、entry、commit、apply、message、index、stroage各个元素之间的关系？


**etcd常用调试命令**
1、查看leader和follower状态
``` shell
# ETCDCTL_API=3 etcdctl endpoint status --cluster -w table
# ./etcdctl endpoint status -w table --endpoints=http://127.0.0.1:12380,http://127.0.0.1:22380,http://127.0.0.1:32380
// 查看API版本
# ./etcdctl version
// 设置为V3版本
# export ETCDCTL_API=3
// V3版本，通过restful API接口设置kv, 需要将kv值转换为base64才能运行，foo --> Zm9v, bar --> YmFy
# curl -L http://localhost:2379/v3beta/kv/put -X POST -d '{"key": "Zm9v", "value": "YmFy"}'
// V2版本，查看hello键的值
# curl http://localhost:2379/v2/keys/hello

```