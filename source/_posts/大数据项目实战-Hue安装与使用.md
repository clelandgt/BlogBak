---
title: 大数据项目实战-Hue安装与使用
date: 2018-07-25 00:00:48
tags:
---

- 基于Django实现
- 官方文档比较乱


## 下载Hue

```
$ wget http://archive.cloudera.com/cdh5/cdh/5/hue-3.9.0-cdh5.5.0.tar.gz
$ tar xvf hue-3.9.0-cdh5.5.0.tar.gz -C /opt/modules
$ yum install ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi gcc gcc-c++ krb5-devel libtidy libxml2-devel libxslt-devel openldap-devel python-devel sqlite-devel openssl-devel mysql-devel gmp-devel	 # 安装依赖
$ make apps # 编译
```

配置desktop/conf/hue.ini文件

```
secret_key=jFE93j;2[290-eiw.KEiwN2s3['d;/.q[eIW^y#e=+Iei*@Mn<qW5o
http_host=header
http_port=8888
time_zone=Asia/Shanghai
```

修改目录权限

```
$ sudo chmod o+w desktop/decktop.db
$ sudo chown hadoop:hadoop desktop/decktop.db
$ sudo chown -R hadoop:hadoop logs
```

启动Hue, 初始设置账户为: user: admin, password: admin

```
$ build/env/bin/supervisor
```

## Hue与HDFS集成

编辑desktop/conf/hue.ini文件

```
fs_defaultfs=header
webhdfs_url=http://header:50070/webhdfs/v1
hadoop_conf_dir=/opt/modules/hadoop-2.5.0/etc/hadoop
hadoop_bin=/opt/modules/hadoop-2.5.0/bin
hadoop_hdfs_home=/opt/modules/hadoop-2.5.0
```

编辑header机器上的$HADOOP_CONF/core-site文件， 分发到worker-1和worker-2

```
	<property>
	    <name>hadoop.proxyuser.hue.hosts</name>
	    <value>*</value>
	</property>
	<property>
	    <name>hadoop.proxyuser.hue.groups</name>
	    <value>*</value>
	</property>
```

然后重启Hue,查看网页上是否可以查看hdfs


## Hue与Yarn集成

编辑desktop/conf/hue.ini文件

```
[[yarn_clusters]]
	[[default]]
resourcemanager_host=header
resourcemanager_port=8032
submit_to=True
resourcemanager_api_url=http://header:8088
proxy_api_url=http://header:8088
history_server_api_url=http://header:19888
```

## Hue与Hive集成

编辑desktop/conf/hue.ini文件

```
[beeswax]
hive_server_host=header
hive_server_port=10000
hive_conf_dir=/opt/modules/hive-0.13.1-cdh5.3.0/conf
```

启动hiveserver2

## Hue与mysql集成
编辑desktop/conf/hue.ini文件

```
[[[mysql]]]
nice_name="My SQL DB"
name=metastore
engine=mysql
host=header
port=3306
user=admin
password=admin
```

## 与hbase集成

启动thrift服务： 

```
$ bin/hbase-daemon.sh start thrift
```
编辑desktop/conf/hue.ini文件

```
hbase_clusters=(Cluster|header:9090)
hbase_conf_dir=/opt/modules/hbase-0.98.6-hadoop2/conf
```

