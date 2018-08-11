---
title: 大数据实战项目-sparksql的使用与配置
date: 2018-08-01 08:32:43
tags:
---

## spark sql与Hive集成(spark-shell)

拷贝hive的配置文件hive-site.xml到spark的conf目录，并保证配置metastore.

```
<property>
    <name>hive.metastore.urls</name>
    <value>thrift://header:9083</value>
</property>
```

拷贝hive的mysql jar包到spark的jar目录
```
$ cp /opt/modules/hive-0.13.1-cdh5.3.0/lib/mysql-connector-java-5.1.7-bin.jar /opt/modules/spark-2.2.0-bin-hadoop2.5.0-spark/jars/
```

编辑spark-env.sh编辑环境变量
```
HADOOP_CONF_DIR=/opt/modules/hadoop-2.5.0
```