---
title: 2019-12-25 kafka
tags: kafka, kafka-client
renderNumberedHeading: true
grammar_cjkRuby: true
---


**kafka-client的使用**
1、调试环境搭建：
1.1）搭建单机版的kafka集群。
        修改配置文件：broker.id、logs.dirs、zookeeper.connect（建议带上路径如/kafka_go表示kafka在zk的根目录）		
``` shell
broker.id=1
log.dirs=/data/xxx/tool/kafka_2.12-2.4.0/logs
zookeeper.connect=localhost:2181/kafka_go
```
1.2）配置zookeeper。
1.3）启动kafka服务。
``` shell
#  nohup bin/kafka-server-start.sh config/server.properties &
```
1.4）创建主题、查看主题、发送消息、启动消费者、搭建多个broker
``` shell
// 创建主题gotest，指定一个分区和一个副本
# bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic gotest
// 查看topic
# bin/kafka-topics.sh --list --bootstrap-server localhost:9092
// 发送消息到主题gotest
# bin/kafka-console-producer.sh --broker-list localhost:9092 --topic gotest
// 启动一个消费者从主题gotest消费消息，指定从头开始消费所有消息
# bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic gotest --from-beginning
// 启动多个broker，搭建kafka集群
// 注意：每台机器的broker.id不能相同，多个机器ip如何识别在同一个集群中？
```
1.5）运行go客户端测试用例（github.com\!shopify\sarama@v1.24.1\examples），检查连接是否正常、生产者和消费者（单消费者模式）功能是否正常。
1.5.1）检查broker是否已成功写入消息，多次运行会写消息到topic，发现topic的offset在不断增加（正常）
``` shell
# ./bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic default_topic
```
![enter description here](./images/1577259176852.png)
1.5.2）检查当前消费者已读的偏移量
``` shell
// 查看某个消费者group下已读的偏移量
# bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
// 查看所有的消费者group
# bin/kafka-consumer-groups.sh  --list --bootstrap-server localhost:9092
```
1.5.3）直接查看kafka消费者提交偏移量的文件内容
官网描述offset：https://kafka.apache.org/documentation/#quickstart
对于单个消费者(standalone consumer)，kafka没有存储对应的偏移量？
``` shell
# ./bin/kafka-dump-log.sh --files /data/xxx/tool/kafka_2.12-2.4.0/data/default_topic-0/00000000000000000000.log
```
1.5.4）如果每条消息提交偏移量是否会很频繁？测试发现是通过定时提交偏移量实现的，默认每秒提交一次偏移量。可以通过修改偏移量提交时间的配置来验证是否根据时间间隔来提交偏移量？
``` golang
// github.com\!shopify\sarama\config.go
c.Consumer.Offsets.CommitInterval = 1 * time.Second
```


**kafka客户端类图**
1、根据"github.com/Shopify/sarama" 部分模块代码进行整理。
2、kafka客户端实例是不是协程安全的，一个进程每个协程可以并发调用？一个进程一般支持多少个客户端连接？


**消费者和消费组**
2、单消费者和消费组的区别：
2.1）单消费者（standalone consumer）没法查看客户端当前提交的偏移量？每次客户端重启后都得从指定位置开始重新消费topic的消息？
2.2）多个单消费者消费同一个topic，会单独消费topic的所有内容，不是共享topic内容。
2.3）消费组可以通过kafka-consumer-groups.sh脚本查看每个消费者提交的偏移量。如果客户端不处理分区消息，LAG会越来越大，表示消息出现堆积。
![enter description here](./images/1577328435487.png)
2.4）消费组里多个消费者消费同一个topic，会共享消费topic的内容。每个消费者消费的消息都不一样。
2.5）消费组里的消费者不处理消息，LAG出现了堆积，并且会出现 no active member的提示，看不到consumer id和client id？
![enter description here](./images/1577329272078.png)

**生产者和消费者模式**
2、kafka生产者和消费者的模式测试：
2.1）生产者模式：
2.2）消费者模式：
2.2.1）单次提交模式
“github.com/Shopify/sarama” 客户端提供了单次提交偏移量的方式ConsumeClaim，循环每次从channel阻塞读取消息，如果不手动提交偏移量（通过MarkMessage），下次会读取到同一条消息，每消费1条消息提交一次偏移量对性能有影响，需要转换为批量提交的方式？
测试方法：
第一步：启动消费组客户端进程，等待消费消息，ExampleConsumerGroup()注释提交偏移量的调用：MarkMessage。
![enter description here](./images/1577346104235.png)
第二步：通过kafka控制台命令，手动发送写kafka消息命令，每次写入不同的kafka消息内容：
![enter description here](./images/1577345645984.png)
第三步：通过kafka控制台命令，检查消费组的偏移量是否发生变化，终端打印的消息内容是否发生变化来确认是否每次只提交一次偏移量。
消费组客户端进程把写入的消息都读取出来了：
![enter description here](./images/1577345576031.png)
消费组已提交的偏移量没有发生变化，LAG堆积的消息在增加：
![enter description here](./images/1577345868824.png)
2.2.2）批量提交模式
参考单次提交模式，“github.com/Shopify/sarama” 客户端可以通过控制MarkMessage调用的频率，来实现批量提交偏移量的功能。
测试方法：
第一步：启动消费组客户端进程，ExampleConsumerGroup控制读取消息的数量来手动提交偏移量：
![enter description here](./images/1577347176041.png)
第二步：通过kafka控制台命令，手动发送写消息命令，写入不同的kafka消息内容，并且观察消费组的偏移量是否发生变化。
第三步：满足条件触发手动提交偏移量，然后再检查消费组的偏移量的变化。
![enter description here](./images/1577347062310.png)


**kafka控制常用调试命令**
3、kafka常用调试命令：查看topic的消费者列表、查看消费者已读偏移量、查看topic的分区数、查看消费者的状态、查看分区是否平衡、查看topic是否有副本
	kafka常用命令可以直接运行/data/xxx/tool/kafka_2.12-2.4.0/bin目录下的脚本，然后根据提示编辑正确的命令。
	github仅供参考：https://gist.github.com/sonhmai/5b2b4455162c808c091b661aeb675625
``` shell
// 查看topic的消费者列表
# 
// 查看消费者已读偏移量
# 
// 查看topic的分区数、副本
# ./bin/kafka-topics.sh --bootstrap-server localhost:9092  --describe --topic default_topic
// 查看消费者的状态
# 
// 查看分区是否平衡
# 

```