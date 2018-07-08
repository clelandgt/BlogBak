---
title: 大数据项目实战-hadoop高可用ha架构
date: 2018-07-06 23:15:52
tags:
---

## HDFS高可用ha

编辑hdfs-site.xml如下：

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>

    <property>
        <name>dfs.nameservices</name>
        <value>ns</value>
    </property>

    <property>
        <name>dfs.ha.namenodes.ns</name>
        <value>nn1,nn2</value>
    </property>

    <property>
        <name>dfs.namenode.rpc-address.ns.nn1</name>
        <value>header:8020</value>
    </property>

    <property>
        <name>dfs.namenode.rpc-address.ns.nn2</name>
        <value>worker-1:8020</value>
    </property>

    <property>
        <name>dfs.namenode.http-address.ns.nn1</name>
        <value>header:50070</value>
    </property>

    <property>
       <name>dfs.namenode.http-address.ns.nn2</name>
       <value>worker-1:50070</value>
    </property>

    <property>
       <name>dfs.namenode.shared.edits.dir</name>
       <value>qjournal://header:8485;worker-1:8485;worker-2:8485/ns</value>
    </property>

    <property>
       <name>dfs.journalnode.edits.dir</name>
       <value>/opt/modules/hadoop-2.5.0/data/jn</value>
    </property>

    <property>
       <name>dfs.client.failover.proxy.provider.ns</name>
       <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>

    <property>
       <name>dfs.ha.fencing.methods</name>
       <value>sshfence</value>
    </property>

    <property>
       <name>dfs.ha.fencing.ssh.private-key-files</name>
       <value>/home/hadoop/.ssh/id_rsa</value>
    </property>

    <property>
       <name>dfs.ha.automatic-failover.enabled.ns</name>
       <value>true</value>
    </property>

    <property>
       <name>ha.zookeeper.quorum</name>
       <value>header:2181,worker-1:2181,worker-2:2181</value>
    </property>

</configuration>
```

## YARN高可用ha

编辑yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>header:8088</value>
    </property>

    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>

    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>-1</value>
    </property>

    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>rm</value>
    </property>

    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>header</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>worker-1</value>
    </property>

    <property>
       <name>yarn.resourcemanager.zk-address</name>
       <value>header:2181,worker-1:2181,worker-2:2181</value>
    </property>

    <property>
       <name>yarn.resourcemanager.recovery.enabled</name>
       <value>true</value>
    </property>

    <property>
       <name>yarn.resourcemanager.store.class</name>
       <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>

</configuration>
```

## 运行与测试
将修改后的配置推到worker-1和worker-2上(注意如何是拷贝hadoop过去需要修改zookeeper-3.4.5/zkData/mypid)，部署方案：

|header|worker-1|worker-2|
|:--|:--|:--|
|namenode|namenode||
|datanode|datanode||
|journalnode|journalnode|journalnode|
|zkfc|zkfc||
|resourcemanager|resourcemanager||
|nodemanager|nodemanager|nodemanager|


启动zookeeper:

```
$ bin/zkServer.sh start  # header
$ bin/zkServer.sh start  # worker-1
$ bin/zkServer.sh start  # worker-2
```


启动journalnode

```
$ sbin/hadoop-daemon.sh start journalnode # header
$ sbin/hadoop-daemon.sh start journalnode # worker-1
$ sbin/hadoop-daemon.sh start journalnode # worker-2
```

格式化hdfs(初始化时使用)

```
$ bin/hdfs namenode -format
```

初始化HA在zookeeper中的状态

```
$ bin/hdfs zkfc -formatZK  # header
```

启动hdfs

```
$ sbin/hadoop-daemon.sh start namenode  # header
$ bin/hdfs namenode -bootstrapStandby  # worker-1, 同步nn1的元数据信息
$ sbin/hadoop-daemon.sh start namenode  # worker-1 
$ sbin/hadoop-daemon.sh start zkfc # header 那台机器启动，那个namenode就active
$ sbin/hadoop-daemon.sh start datanode  # header
$ sbin/hadoop-daemon.sh start datanode  # worker-1
$ sbin/hadoop-daemon.sh start datanode  # worker-2
```

启动yarn

```
$ sbin/yarn-daemon.sh start resourcemanager  # header
$ sbin/yarn-daemon.sh start resourcemanager  # worker-1
$ sbin/yarn-daemon.sh start nodemanager  # header
$ sbin/yarn-daemon.sh start nodemanager  # worker-1
$ sbin/yarn-daemon.sh start nodemanager  # worker-2
$ bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount hdfs://header/data   /user/hadoop/output/1  # 启动测试程序
```


## 异常
```
$ hdfs namenode -format
18/07/07 23:02:15 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = header/172.18.179.240
...
retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
18/07/07 23:02:28 WARN namenode.NameNode: Encountered exception during format:
org.apache.hadoop.hdfs.qjournal.client.QuorumException: Unable to check if JNs are ready for formatting. 1 exceptions thrown:
172.18.179.240:8485: Call From header/172.18.179.240 to header:8485 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
	at org.apache.hadoop.hdfs.qjournal.client.QuorumException.create(QuorumException.java:81)
	at org.apache.hadoop.hdfs.qjournal.client.QuorumCall.rethrowException(QuorumCall.java:223)
	at org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager.hasSomeData(QuorumJournalManager.java:232)
	at org.apache.hadoop.hdfs.server.common.Storage.confirmFormat(Storage.java:875)
	at org.apache.hadoop.hdfs.server.namenode.FSImage.confirmFormat(FSImage.java:171)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:922)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1354)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1473)
18/07/07 23:02:28 FATAL namenode.NameNode: Exception in namenode join
org.apache.hadoop.hdfs.qjournal.client.QuorumException: Unable to check if JNs are ready for formatting. 1 exceptions thrown:
172.18.179.240:8485: Call From header/172.18.179.240 to header:8485 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
	at org.apache.hadoop.hdfs.qjournal.client.QuorumException.create(QuorumException.java:81)
	at org.apache.hadoop.hdfs.qjournal.client.QuorumCall.rethrowException(QuorumCall.java:223)
	at org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager.hasSomeData(QuorumJournalManager.java:232)
	at org.apache.hadoop.hdfs.server.common.Storage.confirmFormat(Storage.java:875)
	at org.apache.hadoop.hdfs.server.namenode.FSImage.confirmFormat(FSImage.java:171)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.format(NameNode.java:922)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.createNameNode(NameNode.java:1354)
	at org.apache.hadoop.hdfs.server.namenode.NameNode.main(NameNode.java:1473)
```

解决方法：先用./zkServer.sh start 启动各个zookeeper，再用./ hadoop-daemon.sh start journalnode启动各个DataNode上的 JournalNode进程。然后再进行格式化即可。


## 参考
- [HDFS High Availability Using the Quorum Journal Manager](http://hadoop.apache.org/docs/r2.5.2/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)
- [ResourceManager High Availability](http://hadoop.apache.org/docs/r2.5.2/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html)