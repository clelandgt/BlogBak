---
title: 大数据项目实战-spark的安装与使用
date: 2018-07-29 15:09:42
tags:
---

<!-- MarkdownTOC -->

- 下载安装java8
- 下载安装scala
- 下载并配置Maven
- 下载并编译Spark

<!-- /MarkdownTOC -->



spark 编译配置java8和maven3.3.9 Spark 源码编译的3种方式

1. Maven编译
2. SBT编译
3. spark官方提供的打包编译make-distribution.sh(编译并打包)

## 下载安装java8
因为spark2.2.0需要java8，所以再次安装java8(java8向下兼容java7)

```
$ wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz
$ tar xvf jdk-8u11-linux-x64.tar.gz -C /opt/modules
```

编辑/etc/profile修改JAVA_HOME

```
export JAVA_HOME=/opt/modules/jdk1.8.0_11
export PATH=$PATH:$JAVA_HOME/bin
```

然后重新加载环境变量

```
$ source /etc/profile
```

## 下载安装scala

```
$ tar xvf scala-2.11.8.tgz -C /opt/modules
```

编辑/etc/profile修改SCALA_HOME添加环境变量

```
# SCALA_HOME
export SCALA_HOME=/opt/modules/scala-2.11.8
export PATH=$PATH:SCALA_HOME/bin
```


## 下载并配置Maven

```
$ wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.5.2-bin.tar.gz
$ tar xvf apache-maven-3.3.9-bin.tar.gz -C /opt/modules/
```
配置Maven环境变量，编辑/etc/profile

```
export MAVEN_HOME=/opt/modules/apache-maven-3.3.9
export PATH=$PATH:$MAVEN_HOME/bin
export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=1024M -XX:ReservedCodeCacheSize=1024m"
```

在/etc/recolv.conf中添加以下内容

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```


## 下载并编译Spark

```
$ wget https://archive.apache.org/dist/spark/spark-2.2.0/spark-2.2.0.tgz
$ tar xvf spark-2.2.0.tgz -C /opt/modules/
```

配置编译脚本dev/make-distribution.sh

```
VERSION=2.2.0  # 当前spark版本
SCALA_VERSION=2.11.8 # scala版本
SPARK_HADOOP_VERSION=2.5.0 # 当前hadoop版本
SPARK_HIVE=1 # 支持hive
```

编译spark源码包

```
$ ./dev/make-distribution.sh --name hadoop2.5.0-spark --tgz  -Phadoop-2.5 -Phive -Phive-thriftserver -Pyarn
```