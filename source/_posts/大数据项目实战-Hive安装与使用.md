---
title: 大数据实战项目-Hive安装与使用
date: 2018-07-23 00:38:18
tags:
---

## 安装mysql

```
$ sudo rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
$ sudo yum install mysql-server
$ sudo service mysqld start  # 启动mysql
$ mysqladmin -u root -h header  password 'ganZHEyu'  # 初始化数据
```

## 安装配置hive

```
$ wget http://archive.cloudera.com/cdh5/cdh/5/hive-0.13.1-cdh5.3.0.tar.gz  # 下载hive
$ tar xvf hive-0.13.1-cdh5.3.0.tar.gz -C /opt/modules/   # 解压hive
$ hadoop fs -mkdir -p /user/hive/warehouse  # hdfs上创建warehouse目录
$ hadoop fs -chmod g+w  /user/hive/warehouse  # hdfs上修改warehouse的权限
$ cp conf/hive-env.sh.template conf/hive-env.sh
```

编辑hive-env.sh

```
HADOOP_HOME = /opt/modules/hadoop-2.5.0
HIVE_CONF_DIR = /opt/modules/hive-0.13.1-cdh5.3.6/conf
```

新建并编辑hive-site文件

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
	    <name>javax.jdo.option.ConnectionURL</name>
	    <value>jdbc:mysql://header:3306/hive?createDatabaseIfNotExist=true</value>
	</property>
	<property>
	    <name>javax.jdo.option.ConnectionDriverName</name>
	    <value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
	    <name>javax.jdo.option.ConnectionUserName</name>
	    <value>root</value>
	</property>
	<property>
	    <name>javax.jdo.option.ConnectionPassword</name>
	    <value>ganZHEyu</value>
	</property>
	<property>
	    <name>hbase.zookeeper.quorum</name>
	    <value>header,worker-1,worker-2</value>
	</property>	
</configuration>
```

拷贝hive连接mysql

```
$ cp mysql-connector-java-5.1.7-bin.jar /opt/modules/hive-0.13.1-cdh5.3.0/lib
```

mysql中设置hive用户

```
mysql> use mysql;
mysql> select User, Host, Password from user;
mysql> update user set Host='%' where User='root' and Host='localhost';
mysql> delete from user where user='root' and host='127.0.0.1';
mysql> delete from user where user='root' and host='header';
mysql> delete from user where host='localhost';
mysql> flush privileges;  -- 刷新权限
```

## hive集成hbase

将hbase的这九个包拷贝到hive/lib下，如果CDH同版本，就不需要拷贝，因为CDH本身已经做了集成。

- hbase-client-0.98.6-hadoop2.jar
- hbase-it-0.98.6-hadoop2.jar
- htrace-core-2.04.jar
- hbase-common-0.98.6-hadoop2.jar
- hbase-protocol-0.98.6-hadoop2.jar
- hbase-hadoop2-compat-0.98.6-hadoop2.jar
- hbase-server-0.98.6-hadoop2.jar
- hbase-hadoop-compat-0.98.6-hadoop2.jar
- high-scale-lib-1.1.1.jar


创建业务表结构

``` mysql
CREATE EXTERNAL TABLE weblogs(
	id STRING,
	datetime STRING,
	userid STRING,
	searchname STRING,
	retorder STRING,
	cliorder STRING,
	cliurl STRING
)
STORED BY "org.apache.hadoop.hive.hbase.HBaseStorageHandler"
WITH SERDEPROPERTIES("hbase.columns.mapping"=":key,info:datetime,info:userid,info:searchname,info:retorder,info:cliorder,info:cliurl")
TBLPROPERTIES("hbase.table.name"="weblogs");
```

