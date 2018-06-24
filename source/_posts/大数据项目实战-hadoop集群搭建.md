---
title: 大数据项目实战-hadoop集群搭建
date: 2018-06-25 00:15:45
tags:
---

主机清单

|角色|外网ip|内网ip|
|:--|:--|:--|:--|
|header|120.77.44.77|172.18.179.240|
|worker-1|39.108.184.123|172.18.179.241|
|worker-2|39.108.1.16|172.18.179.242|


账户信息都是：

- root: M2J*b7AKjkaZi]Hw
- hadoop: ganZHEyu

## linux初始配置
### 添加hadoop用户

```
$ adduser hadoop
$ passwd hadoop  # 密码为ganZHEyu
```
声明可以使用hadoop用户可以使用任何命令且不使用密码

```
$ su -   # 输入密码切换为root切换
$ vim /etc/sudoers  # 编辑该文件，
1. 在“root ALL=(ALL)ALL"行下面添加"xxx ALL=(ALL)ALL"，其中xxx是你要添加的用户名hadoop。因为该文件是只读的，所以输入":wq!"退出。
2. 添加“hadoop  ALL=(root)NOPASSWD:ALL”
```

TODO: 待了解sudoers

### 修改hostname与地址映射

依次修改host名: header, worker-1, worker-2。

```
$ hostnamectl set-hostname header 
```
添加地址映射

```
$ vim /etc/hosts  # 添加以下内容
172.18.179.240  header
172.18.179.241  worker-1
172.18.179.242  worker-2

$ reboot # 重启生效
```

### 免密登录设置
配置header免密登录worker-1, worker-2

```
$ ssh-keygen  # 生成秘钥
$ ssh-copy-id hadoop@worker-1  # 输入密码，推送公钥到worker-1
$ ssh-copy-id hadoop@worker-2  # 输入密码，推送公钥到worker-2
```

### 安装java
在/opt目录下创建：softwares, tools, modules, datas四个目录

```
$ cd /opt/software
$ tar -xzf jdk-7u67-linux-x64.tar.gz /opt/modules/ # 安装java
```

添加java环境变量，编辑文件/etc/profile， 末尾添加以下内容

```
# JAVA_HOME
export JAVA_HOME=/opt/modules/jdk1.7.0_67
export PATH=$PATH:$JAVA_HOME/bin
```

## Hadoop部署
选择2.5以上hadoop版本,下载的地方：

http://archive.apache.org/dist/
http://archive.cloudera.com/cdh5/

### 下载hadoop安装包并解压


```
$ cd /opt/softwares/
$ tar -xzf hadoop-2.5.0.tar.gz -C /opt/modules/
```

### 配置hdfs

#### 配置java环境变量
在hadoop-env.sh,mapred-enc.sh,yarn-env.sh3个文件中添加java环境变量：


```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```

#### 配置core-site.xml
设置dfs.replication=2，副本数为2，默认为3.

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
</configuration>
```

#### 配置hdfs-site.xml

- 配置hdfs网络服务为hdfs://header:9000；
- 配置用户为hadoop；
- 配置日志目录为/opt/modules/hadoop-2.5.0/data/tmp, 修改了该地方，hdfs需要重新初始化。


```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://header:9000</value>
    </property>

    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>hadoop</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/modules/hadoop-2.5.0/data/tmp</value>
    </property>
</configuration>
```
#### 配置slaves

```
header
worker-1
worker-2
```

#### 启动服务

```
$ bin/hdfs namenode -format # 格式化文件系统hdfs
$ sbin/hadoop-daemon.sh start namenode  # 启动namenode
$ sbin/hadoop-daemon.sh start datanode  # 启动datanode, header上也规划了一个datanode
$ jps # 查看服务是否都正常打开
4405 Jps
3253 DataNode
2707 NameNode
```

输入 http://header:50070/访问hdfs管理页面。拷贝配置好的hadoop到worker-1, worker-2。

```
$ scp -r hadoop-2.5.0 hadoop@worker-1:/opt/modules
$ scp -r hadoop-2.5.0 hadoop@worker-2:/opt/modules
```

启动worker-1, worker-2上的datanode

```
$ sbin/hadoop-daemon.sh start datanode
```

### 配置yarn

配置yarn,编辑文件yarn-site.xml添加

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>header</value>
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

</configuration>
```

配置mapreduce,编辑文件mapred-site.xml添加

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>header:10020</value>
    </property>

    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>header:19888</value>
    </property>

</configuration>
```

拷贝配置好的hadoop到worker-1, worker-2.然后启动相应服务看yarn和mapreduce是否配置正常

```
$ sbin/yarn-daemon.sh start nodemanager
$ sbin/yarn-daemon.sh start resourcemanager
$ sbin/mr-jobhistory-daemon.sh start historyserver
$ jps  # 查看服务是否都正常打开
3706 JobHistoryServer
4405 Jps
3253 DataNode
4012 NodeManager
4142 ResourceManager
2707 NameNode
```

输入 http://header:8088/访问yarn管理页面，添加worker-1, worker-2到集群yarn,分别在worker-1, worker-2下执行：

```
$ sbin/yarn-daemon.sh start nodemanager
```

### 启动服务
#### header

```
$ bin/hdfs namenode -format
$ sbin/hadoop-daemon.sh start namenode
$ sbin/hadoop-daemon.sh start datanode
$ sbin/yarn-daemon.sh start nodemanager
$ sbin/yarn-daemon.sh start resourcemanager
$ sbin/mr-jobhistory-daemon.sh start historyserver
```

#### worker

```
$ sbin/hadoop-daemon.sh start datanode
$ sbin/yarn-daemon.sh start nodemanager
```

## 其他
### 阿里云安全组
使用阿里云时，需要注意使用安全组，配置上需要开放的端口。

## 问题
### 1. 无法启动datanode,报错"ulimit -a for user root"
datanamenode运行时打开文件数，达到系统最大限制

```
$ ulimit -n   # 查看当前最大限制
65536
$ vim /etc/security/limits.conf  # 修改nofile限制
```

加上：

```
* soft nodefile 102400
* hard nodefile 409600
```

还不能解决，就清除hadoop.tmp.dir对应目录下对应的文件。

### 2. hdfs 显示本地目录问题

TODO: 暂时使用bin/hadoop fs -ls hdfs://header:9000/方式访问


