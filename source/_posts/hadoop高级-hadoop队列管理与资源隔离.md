---
title: hadoop高级-hadoop队列管理与资源隔离
date: 2018-05-17 20:13:01
tags: hadoop
---

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive-hive使用压缩/hive-hive使用压缩1.jpg)

<!-- more -->

<!-- MarkdownTOC -->

- 功能点
- 配置
  - 配置ResouceManager使用CapacityScheduler
  - 设置队列
  - 队列属性
    - 队列属性配置
    - 修改配置文件
  - 设置任务的优先级
- 其他组件使用hadoop队列
  - Hive
- 补充
- 参考

<!-- /MarkdownTOC -->


## 功能点
- Hierarchical Queues(队列可分层)
- Capacity Guarantees
- Security(安全性)
- Elasticity
- Multi-tenancy
- Operability
- Resource-based Scheduling
- Queue Mapping based on User or Group: 可以基于用户或用户组分配队列；
- Priority Scheduling(优先级): 

## 配置
### 配置ResouceManager使用CapacityScheduler
修改`conf/yarn-site.xml`文件

|属性|值|
|:--|:--|
|yarn.resourcemanager.scheduler.class|org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler| 

### 设置队列
修改`capacity-scheduler.xml`配置文件.

|属性|值|
|:--|:--|
|yarn.scheduler.capacity.root.queues|default, batch | 

``` xml
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,batch</value>
    <description>The queues at the this level (root is the root queue).</description>
  </property>
```

 其中默认有default队列，现添加queues11队列。

### 队列属性
- yarn.scheduler.capacity.root.default.capacity：一个百分比的值，表示占用整个集群的百分之多少比例的资源，这个queue-path下所有的capacity之和是100
- yarn.scheduler.capacity.root.default.user-limit-factor：每个用户的低保百分比，比如设置为1，则表示无论有多少用户在跑任务，每个用户占用资源最低不会少于1%的资源
- yarn.scheduler.capacity.root.default.maximum-capacity：弹性设置，最大时占用多少比例资源
- yarn.scheduler.capacity.root.default.state：队列状态，可以是RUNNING或STOPPED
- yarn.scheduler.capacity.root.default.acl_submit_applications：哪些用户或用户组可以提交任务
- yarn.scheduler.capacity.root.default.acl_administer_queue：哪些用户或用户组可以管理队列

#### 队列属性配置
- 资源分配比例: default(40%), batch(60%)
- 队列管理权限: default与batch队列管理员为admin
- 队列提交任务: detault队列为所有用户均可提交任务，batch队列只有devops, admin用户可以提交任务。 
- default队列里单个任务最大占用资源比例为60%

#### 修改配置文件

``` xml
  <!-- default queue -->
  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
    <description>Default queue target capacity.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.user-limit-factor</name>
    <value>1</value>
    <description>Default queue user limit a percentage from 0.0 to 1.0.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
    <value>60</value>
    <description>The maximum capacity of the default queue.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.state</name>
    <value>RUNNING</value>
    <description>The state of the default queue. State can be one of RUNNING or STOPPED.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.acl_submit_applications</name>
    <value>*</value>
    <description>The ACL of who can submit jobs to the default queue.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.acl_administer_queue</name>
    <value>admin</value>
    <description>The ACL of who can administer jobs on the default queue.</description>
  </property>
  

  <!-- batch queue -->
  <property>
    <name>yarn.scheduler.capacity.root.batch.capacity</name>
    <value>60</value>
    <description>batch queue target capacity.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.batch.user-limit-factor</name>
    <value>1</value>
    <description>batch queue user limit a percentage from 0.0 to 1.0.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.batch.maximum-capacity</name>
    <value>100</value>
    <description>The maximum capacity of the batch queue.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.batch.state</name>
    <value>RUNNING</value>
    <description>The state of the queues11 queue. State can be one of RUNNING or STOPPED.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.batch.acl_submit_applications</name>
    <value>admin,devops</value>
    <description>The ACL of who can submit jobs to the batch queue.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.batch.acl_administer_queue</name>
    <value>admin</value>
    <description>The ACL of who can administer jobs on the batch queue.</description>
  </property>
```

### 设置任务的优先级

## 其他组件使用hadoop队列
### Hive
```
SET mapreduce.job.queuename=batch;
```

## 补充
- ACLs: DFS支持POSIXAccess Control Lists（ACL），也就是传统的POSIX权限控制。通过提供一种方式来设置特定用户或用户组指定为不同权限的HDFS文件ACL访问控制，类似于Linux文件系统权限.


## 参考
1. [Hadoop: Capacity Scheduler](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html)
2. [HDFS ACLs访问控制权限](https://blog.csdn.net/kimsungho/article/details/51418015)