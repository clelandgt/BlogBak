---
title: 大数据项目实战-Hbase+Kafka+Flume集成安装
date: 2018-07-17 22:59:12
tags:
---
<!-- MarkdownTOC -->

- hbase
    - 下载安装hbase
    - 启动hbase
    - HBase常用命令
        - 表操作命令
- Kafka
    - 下载安装kafka
    - 启动测试kafka
- Flume
    - 下载安装
- Flume+HBase+Kafka集成
    - 配置header\(聚合节点\)
    - flume改写
        - 下载
        - 修改源码
        - 测试

<!-- /MarkdownTOC -->


## hbase
### 下载安装hbase

```
$ wget http://archive.apache.org/dist/hbase/hbase-0.98.6/hbase-0.98.6-hadoop2-bin.tar.gz
$ tar xvf hbase-0.98.6-hadoop2-bin.tar.gz -C /opt/modules/  # 解压到指定目录
```

编辑hbase-env.sh文件

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
export HBASE_MANAGES_ZK=false
```

编辑hbase-site.xml文件

```
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://header:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>header,worker-1,worker-2</value>
  </property>
```

编辑regionservers文件

```
header
worker-1
worker-2
```

新建backup-masters文件,添加需要用来作为备份的主机

```
worker-1
```

### 启动hbase
```
$ bin/hbase-daemon.sh start master  # header
$ bin/hbase-daemon.sh start regionserver # header
$ bin/hbase-daemon.sh start regionserver # worker-1
$ bin/hbase-daemon.sh start regionserver # worker-2
```

输入http://header:60010/访问hbase管理界面

```
$ hbase shell
```


### HBase常用命令
#### 表操作命令
- create 
- describe
- is_enabled
- drop 
- enable
- is_disabled
- disable
- list
- count
- delete
- get
- put
- scan
- truncate

```
hbase> help  # 查看命令
hbase> create 'test', 'info'
hbase> put 'test','0001','info:userName','laocao'
hbase> put 'test','0001','info:age','20'
hbase> put 'test','0001','info:tel','13500000000'
hbase> put 'test','0002','info:userName','小明'
hbase> put 'test','0002','info:age','22'
hbase> put 'test','0002','info:tel','13500000002'
hbase> scan 'test'
hbase> count 'test'
hbase> disable 'test'
hbase> drop 'test'
```

## Kafka
### 下载安装kafka
```
$ wget https://archive.apache.org/dist/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz
$ tar xvf kafka_2.11-0.8.2.1.tgz -C /opt/modules
$ mkdir kafka-logs  # kafka主目录下创建文件夹
```


在header上编辑config/server.properties文件

```
broker.id=0
host.name=header
log.dirs=/opt/modules/kafka_2.11-0.8.2.1/kafka-logs
zookeeper.connect=header:2181,worker-1:2181,worker-2:2181
```

分发服务到worker-1, worker-2，修改server.properties文件的broker.id分别为1，2和host.name与主机名对应。

### 启动测试kafka
```
$ bin/kafka-server-start.sh config/server.properties  # 启动kafka服务
$ bin/kafka-topics.sh --create --zookeeper header:2181,worker-1:2181,worker-2:2181 --replication-factor 1 --partitions 1 --topic protest  # 创建好topic
```

 - --replication-factor 为副本数
 - paritions 分区数：后面性能调优可以使用

查看创建的topics

```
$ bin/zkCli.sh  # 到zookeeper上去看创建的目录
[zk: localhost:2181(CONNECTED) 4] ls /brokers/topics
[protest]
```

启动生产者和消费者

```
$ bin/kafka-console-producer.sh --broker-list header:9092,worker-1:9092,worker-2:9092 --topic protest # 启动生产者
$ bin/kafka-console-consumer.sh --zookeeper header:2181,worker-1:2181,worker-2:2181 --from-beginning  --topic protest  # 启动消费者
```
在生产者的命令行里输入，消费者实时显示消费。

ps：可以启动三个kafka服务，可把在producer.properties文件中添加3个服务器metadata.broker.list=header:9092,worker-1:9092,worker-2:9092

## Flume
### 下载安装
```
$ wget http://archive.apache.org/dist/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
$ tar xvf apache-flume-1.7.0-bin.tar.gz -C /opt/modules/
```
分发到worker-1, worker-2上。先配置worker-1, 编辑flume-env.sh添加环境变量

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```

编辑flume-conf.properties文件

```
agent.sources = r1
agent.channels = c1
agent.slinks = s1

agent.sources.r1.type = exec
agent.sources.r1.command = tail -F /opt/datas/weblogs.log
agent.sources.r1.channels = c1

agent.channels.c1.type = memory
agent.channels.c1.capacity = 10000
agent.channels.c1.transactionCapacity = 10000
agent.channels.c1.keep-alive=5

agent.sinks.k1.type = avro
agent.sinks.k1.channel = c1
agent.sinks.k1.hostname = header
agent.sinks.k1.port = 4545
```

-  capacity: 容量大小
-  transactionCapacity: 事务最大容量
-  keep-alive: 数据调优，source插入到channel时等待(channel空间满了)，slink从channel读取数据为空时等待。


同理配置worker2编辑flume-env.sh添加环境变量

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```

编辑flume-conf.properties文件

```
agent2.sources = r1
agent2.channels = c1
agent2.slinks = s1

agent2.sources.r1.type = exec
agent2.sources.r1.command = tail -F /opt/datas/weblogs.log
agent2.sources.r1.channels = c1

agent2.channels.c1.type = memory
agent2.channels.c1.capacity = 10000
agent2.channels.c1.transactionCapacity = 10000
agent2.channels.c1.keep-alive=5

agent2.sinks.k1.type = avro
agent2.sinks.k1.channel = c1
agent2.sinks.k1.hostname = header
agent2.sinks.k1.port = 4545
```

## Flume+HBase+Kafka集成
1. 配置flume-conf.properties和flume-env.sh
2. 数据预处理(tab转化为逗号)然后分发到worker-1, worker-2的data
3. 程序二次开发：下载fulme源码，idea点击加载flume-ng-hbase-sink，修改源码编译打包
### 配置header(聚合节点)
同理配置worker2编辑flume-env.sh添加环境变量

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```

编辑flume-conf.properties文件

```
agent1.sources = r1
agent1.channels = kafkaC hbaseC
agent1.slinks = kafkaSink  hbaseSink

#************** flume + hbase *************
agent1.sources.r1.type = avro
agent1.sources.r1.channels = hbaseC
agent1.sources.r1.bind = header
agent1.sources.r1.port = 5555
agent1.sources.r1.threads = 5

agent1.channels.hbaseC.type = memory
agent1.channels.hbaseC.capacity = 10000
agent1.channels.hbaseC.transactionCapacity = 10000
agent1.channels.hbaseC.keep-alive = 20

agent1.sinks.hbaseC.type = asynchbase
agent1.sinks.hbaseC.table = weblogs
agent1.sinks.hbaseC.columnFamily = info
agent1.sinks.hbaseC.serializer = org.apache.flume.sink.hbase.KfkAsyncHbaseEventSerializer
agent1.sinks.hbaseC.channel = hbaseC
agent1.sinks.hbaseC.serializer.payloadColumn=datetime,userid,searchname,retorder,cliorder,cliurl

#************** flume + kafak **************
agent1.kafkaC.hbaseC.type = memory
agent1.kafkaC.hbaseC.capacity = 10000
agent1.kafkaC.channels.hbaseC.transactionCapacity = 10000
agent1.kafkaC.channels.hbaseC.keep-alive = 20

agent1.sinks.kafkaSink.channel = kafkaC
agent1.sinks.kafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
agent1.sinks.kafkaSink.brokerList = header:9092,worker-1:9092,worker-2:9092
agent1.sinks.kafkaSink.topic = test
agent1.sinks.kafkaSink.zookeeperConnect = header:2181,worker-1:2181,worker-2:2181
agent1.sinks.kafkaSink.requiredAcks = 1
agent1.sinks.kafkaSink.batchSize=1
agent1.sinks.kafkaSink.serializer.class = kafka.serializer.StringEncoder
```

相关参数的优化：

- agent1.sinks.kafkaSink.requiredAcks：消息的确认
- agent1.sinks.kafkaSink.batchSize: 给kafka推送批量的msg数
- agent1.sinks.kafkaSink.serializer.class: 解析类

### flume改写
#### 下载

```
$ wget http://archive.apache.org/dist/flume/1.7.0/apache-flume-1.7.0-src.tar.gz
```

#### 修改源码
SimpleRowKeyGenerator类新增函数

```
public static byte[] getKfkRowKey(String userid, String datetime) throws UnsupportedEncodingException {
return (userid + datetime + String.valueOf(System.currentTimeMillis()).getBytes("UTF8");
}
```

新建KfkAsyncHbaseEventSerializer类(拷贝自SimpleAsyncHbaseEventSerializer)，并修改函数getActions

``` java
@Override
public List<PutRequest> getActions() {
    List<PutRequest> actions = new ArrayList<PutRequest>();
    if (payloadColumn != null) {
        byte[] rowKey;
        try {
            String[] columns = String.valueOf(payloadColumn).split(",");
            String[] values = String.valueOf(this.payload).split(",");
            for(int i=0; i<columns.length; i++){
                byte[] colColumn = columns[i].getBytes();
                byte[] colValue = values[i].getBytes();
                String datetime = values[0].toString();
                String userid = values[1].toString();
                rowKey = SimpleRowKeyGenerator.getKfkRowKey(userid, datetime);
                PutRequest putRequest = new PutRequest(table, rowKey, cf, colColumn, colValue);
                actions.add(putRequest);
            }

        } catch (Exception e) {
            throw new FlumeException("Could not get row key!", e);
        }
    }
    return actions;
}
```

编译打包jar包。

#### 测试
