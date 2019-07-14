# Kafka集群配置

## 配置文件（serve.properties）

- **broker.id** 每台节点的id
- **listeners** PLAINTEXT:当前节点IP:端口
- **log.dir** 日志存储目录
- **num.partitions** 每个Topic默认的partition分区数为1
- **zookeeper.connect** zookeeper的连接地址（集群/单机）

## 启动命令

```shell
sh kafka-server-start.sh -daemon ../config/server.properties
```

## Zookeeper节点信息

- cluster
- **controller** 存储当前节点中的leader
- controller_epoch
- **brokers** 存储当前所有kafka节点的详细信息
- admin
- isr_change_notification
- **consumers** 当有消费者注册到zk的时候存储到这个节点上
  - ids
  - owners
  - offset
- latest_producer_id_block
- config

## 创建Topic命令

```sh
sh kafkaTopics.sh --create --zookeeper {ip:port} --replication-factor 1 --partitions 1 --topic {topic_name}
```

## 查看Topic命令

```sh
sh kafka-topics.sh --list --zookeeper {ip:port}
```

## 启动控制台生产者向指定Topic发送消息

```sh
sh kafka-console-producer.sh --broker-list {kafka集群地址} --topic {topic_name}
```

## 启动控制台消费者监听指定Topic发来的消息

```sh
sh kafka-console-consumer.sh --bootstrap-server {ip:port} --topic  {topic_name} --from-beginning
```

# kafka名词解释

topic 逻辑**消息集合**，每一个topic可以有多个生产者向他推送消息

partition 才是真正存放**消息**的区域

kafka中最基本的数据单元是消息，我们可以简单的把消息理解成数据库表中的一条记录。消息是由字符数组组成，每一个消息可以有一个可选的key，这个key也是字符数组。key的作用是来hash后路由消息具体写入到哪一个分区（partition）

# kafka的高吞吐量的因素

1. 每一个partition都是顺序写入数据，省去了随机存储寻址的步骤
2. 批量发送 batch.size（当数据到达一定大小）/linger.ms（经过某一段时长）
3. 零拷贝，直接使用内存进行拷贝，省去了将数据从**内核态**到**用户态**再到**内核态**的过程

# 日志策略

日志保留策略

日志压缩策略

# 生产者发送消息到Broker的三种确认方式

request.required.acks

acks=0，生产者不会等待broker(leader)发送ack

acks=1，生产者会等待**leader**发送ack

acks=-1，生产者会等待**所有的follower**都发送ack

# kafka消息的可靠性

假如kafka集群中有三台机器，编号分别为1，2，3，那么在创建topic的时候，如果设置了分区，即```--partitions {个数}```，那么设置```--replication-factor {副本个数=2}```之后，会在三台机器中两两交叉备份。

- 1机器的日志目录下会有```{topic_name}-1```，```{topic_name}-2```
- 2机器的日志目录下会有```{topic_name}-2```，```{topic_name}-3```
- 3机器的日志目录下会有```{topic_name}-3```，```{topic_name}-1```

# leader选举

ISR（副本同步队列）维护有资格成为leader的follower，条件如下

1. 副本的所有节点都必须要和zookeeper保持连接状态
2. 副本的最后一条消息的offset要和leader副本的最后一条消息的offset的差值不能超过指定的阈值，这个阈值（replica.lag.max.messages）是可以设置的

**HW**，HighWatermark，高水印值，指向leader和副本共同存在最新数据的offset

**LEO**，Last End Offset，日志末端偏移量，指向leader存在的最新数据的offset

# kafka查看日志命令

```sh
sh kafka-run-class.sh kafka.tools.DumpLogSegments --files {kafka.log.path} --print-data-log
```

