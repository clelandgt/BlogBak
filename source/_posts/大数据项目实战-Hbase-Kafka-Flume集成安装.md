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
        - 配置worker-1与worker-2
        - 配置header
    - 修改flume源码
    - 修改flume源码
        - 下载
        - 修改源码
- Flume+HBase+Kafka集成
    - 下载用户查询日志
    - 编写启动脚本
        - 编写运行模型程序的shell脚本
        - 编写flume集群服务启动的脚本
        - 编写kafka消费脚本
    - 启动数据采集所有服务
        - 启动HDFS服务
        - 启动Zookeeper服务
        - 启动Hbase服务
        - 启动Kafka服务
        - 启动Flume-1服务\(worker-1\)
        - 启动Flume-2服务\(worker-2\)
        - 启动Flume-3服务\(header\)
- 问题
    - 关闭hdfs的安全模式
- 引用

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
$ wget https://archive.apache.org/dist/kafka/0.9.0.0/kafka_2.11-0.9.0.0.tgz
$ tar xvf kafka_2.11-0.9.0.0.tgz -C /opt/modules
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
如上，下载flume到worker-1。worker-2, header上。


#### 配置worker-1与worker-2

编辑flume-env.sh添加环境变量

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```

编辑flume-conf.properties文件

```
agent.sources = s1
agent.channels = c1
agent.sinks = k1

agent.sources.s1.type = exec
agent.sources.s1.command = tail -F /opt/datas/weblog-flume.log
agent.sources.s1.channels = c1

agent.channels.c1.type = memory
agent.channels.c1.capacity = 10000
agent.channels.c1.transactionCapacity = 10000
agent.channels.c1.keep-alive=5

agent.sinks.k1.type = avro
agent.sinks.k1.channel = c1
agent.sinks.k1.hostname = header
agent.sinks.k1.port = 55555
```

-  capacity: 容量大小
-  transactionCapacity: 事务最大容量
-  keep-alive: 数据调优，source插入到channel时等待(channel空间满了)，slink从channel读取数据为空时等待。

#### 配置header
编辑flume-env.sh添加环境变量

```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
export HADOOP_HOME=/opt/modules/hadoop-2.5.0
export HBASE_HOME=/opt/modules/hbase-0.98.6-hadoop2
```

编辑flume-conf.properties文件

```
agent.sources = r1
agent.channels = kafkaC hbaseC
agent.sinks = kafkaSink  hbaseSink

agent.sources.r1.type = avro
agent.sources.r1.channels = hbaseC kafkaC
agent.sources.r1.bind = header
agent.sources.r1.port = 55555
agent.sources.r1.threads = 5

#************** flume + hbase *************
agent.channels.hbaseC.type = memory
agent.channels.hbaseC.capacity = 10000
agent.channels.hbaseC.transactionCapacity = 10000
agent.channels.hbaseC.keep-alive = 20

agent.sinks.hbaseSink.type = asynchbase
agent.sinks.hbaseSink.table = weblogs
agent.sinks.hbaseSink.columnFamily = info
agent.sinks.hbaseSink.serializer = org.apache.flume.sink.hbase.KfkAsyncHbaseEventSerializer
agent.sinks.hbaseSink.channel = hbaseC
agent.sinks.hbaseSink.serializer.payloadColumn=datetime,userid,searchname,retorder,cliorder,cliurl

#************** flume + kafak **************
agent.channels.kafkaC.type = memory
agent.channels.kafkaC.capacity = 10000
agent.channels.kafkaC.transactionCapacity = 10000
agent.channels.kafkaC.keep-alive = 20

agent.sinks.kafkaSink.channel = kafkaC
agent.sinks.kafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
agent.sinks.kafkaSink.brokerList = header:9092,worker-1:9092,worker-2:9092
agent.sinks.kafkaSink.topic = weblogs
agent.sinks.kafkaSink.zookeeperConnect = header:2181,worker-1:2181,worker-2:2181
agent.sinks.kafkaSink.requiredAcks = 1
agent.sinks.kafkaSink.batchSize=1
agent.sinks.kafkaSink.serializer.class = kafka.serializer.StringEncoder
```

相关参数的优化：

- agent1.sinks.kafkaSink.requiredAcks：消息的确认
- agent1.sinks.kafkaSink.batchSize: 给kafka推送批量的msg数
- agent1.sinks.kafkaSink.serializer.class: 解析类

### 修改flume源码
### 修改flume源码
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
            String[] columns = new String(this.payloadColumn).split(",");
            String[] values = new String(this.payload).split(",");

            for(int i=0; i<columns.length; i++){
                byte[] colColumn = columns[i].getBytes();
                byte[] colValue = values[i].getBytes();

                if(columns.length != values.length) break;
                String datetime = String.valueOf(values[0]);
                String userid = String.valueOf(values[1]);
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

AsyncHBaseSink类中的修改SimpleAsyncHbaseEventSerializer为KfkAsyncHbaseEventSerializer

```
eventSerializerType =
          "org.apache.flume.sink.hbase.KfkAsyncHbaseEventSerializer";
```

编译打包jar包,上传到header的flume的lib目录。


## Flume+HBase+Kafka集成

### 下载用户查询日志
访问 [搜狗实验室用户查询数据](http://www.sogou.com/labs/resource/q.php), 先选用精简版

在worker-1和worker-2上下载部署查询日志。

```
$ wget http://download.labs.sogou.com/dl/sogoulabdown/SogouQ/SogouQ.mini.tar.gz
$ tar -xvf SogouQ.mini.tar.gz -C /opt/datas
$ cat SogouQ.sample | tr "\t" "," > weblog2.log  # 将文件中的tab更换成逗号
$ cat weblog2.log | tr " " "," > weblogs.log
```

### 编写启动脚本
#### 编写运行模型程序的shell脚本
worker-1和worker-2 /opt/datas/weblog-flume.sh

```
#/bin/bash
echo "start log ...."
java -jar /opt/jar/weblogs.jar  /opt/datas/weblogs.log weblog-flume.log
```

#### 编写flume集群服务启动的脚本

flume主目录下创建文件flume-kfk-start.sh  [worker-1]

```
#/bin/bash
echo "flume-1 start ....."
bin/flume-ng agent --conf conf -f conf/flume-conf.properties -n agent -Dflume.root.logger=DEBUG,console 
```

flume主目录下创建文件flume-kfk-start.sh  [worker-2]

```
#/bin/bash
echo "flume-2 start ....."
bin/flume-ng agent --conf conf -f conf/flume-conf.properties -n agent -Dflume.root.logger=INFO,console 
```

flume主目录下创建文件flume-kfk-start.sh  [header]

```
#/bin/bash
echo "flume-3 start ....."
bin/flume-ng agent --conf conf -f conf/flume-conf.properties -n agent -Dflume.root.logger=INFO,console 
```

#### 编写kafka消费脚本
在header,worker-1,worker-2的kafka主目录下都存放kfk-test-consumer.sh文件

```
#/bin/bash
echo "kfk-kafka-comsumer.sh start ....."
bin/kafka-console-consumer.sh -zookeeper header:2181,worker-1:2181,worker-2:2181 -from-beginning -topic weblogs
```

### 启动数据采集所有服务
#### 启动HDFS服务

```
$ sbin/hadoop-daemon.sh start namenode  # header
$ sbin/hadoop-daemon.sh start datanode  # header
$ sbin/hadoop-daemon.sh start datanode  # worker-1
$ sbin/hadoop-daemon.sh start datanode  # worker-2
```

#### 启动Zookeeper服务

```
$ bin/zkServer.sh start  # header
$ bin/zkServer.sh start  # worker-1
$ bin/zkServer.sh start  # worker-2
$ bin/zkServer.sh status  # 查看状态
```

#### 启动Hbase服务

```
$ bin/hbase-daemon.sh start master  # header
$ bin/hbase-daemon.sh start regionserver # header
$ bin/hbase-daemon.sh start regionserver # worker-1
$ bin/hbase-daemon.sh start regionserver # worker-2
```

创建业务表

```
hbase > create 'weblogs', 'info'
```


#### 启动Kafka服务

```
$ bin/kafka-server-start.sh config/server.properties  # header
$ bin/kafka-server-start.sh config/server.properties  # worker-1
$ bin/kafka-server-start.sh config/server.properties  # worker-2
$ bin/zkCli.sh  # 进入zookeeper客户端
```

删除之前的topic

```
zk> ls /brokers/topics
[protest]
zk> rmr /brokers/topics/protest
```
创建新的topic

```
$ bin/kafka-topics.sh --create --zookeeper header:2181,worker-1:2181,worker-2:2181 --replication-factor 3 --partitions 1 --topic weblogs
```


#### 启动Flume-1服务(worker-1，worker-2)

```
$ ./flume-kfk-start.sh  # worker-1的flume主目录
$ ./weblog-flume.sh  # worker-1的/opt/datas/中写入实时日志
$ ./flume-kfk-start.sh  # worker-2的flume主目录
$ ./weblog-flume.sh  # worker-2的/opt/datas/中写入实时日志
```


#### 启动Flume-3服务(header)

```
$ ./flume-kfk-start.sh  # header的flume主目录
$ ./kfk-test-consumer.sh  # worker-1里的kafka消费程序
```

## 问题
### 关闭hdfs的安全模式

```
$ bin/hadoop dfsadmin -safemode leave
```

### flume+kafka报错
问题：

```
2018-07-22 22:20:33,733 (kafka-producer-network-thread | producer-1) [ERROR - org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:130)] Uncaught error in kafka producer I/O thread:
org.apache.kafka.common.protocol.types.SchemaException: Error reading field 'throttle_time_ms': java.nio.BufferUnderflowException
	at org.apache.kafka.common.protocol.types.Schema.read(Schema.java:71)
	at org.apache.kafka.clients.NetworkClient.handleCompletedReceives(NetworkClient.java:439)
	at org.apache.kafka.clients.NetworkClient.poll(NetworkClient.java:265)
	at org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:216)
	at org.apache.kafka.clients.producer.internals.Sender.run(Sender.java:128)
	at java.lang.Thread.run(Thread.java:745)
```

解决方案：

升级kafka版本为0.9.x


## 引用
- [Hadoop错误9_解决Hadoop的Safe mode is ON问题](https://blog.csdn.net/wang_zhenwei/article/details/48707981)
- [Kafka - Flume - Oracle Database - Error reading field 'throttle_time_ms'](https://stackoverflow.com/questions/43612344/kafka-flume-oracle-database-error-reading-field-throttle-time-ms)