---
title: tableau连接hive
date: 2018-02-05 23:19:38
tags: hive,tableau
---

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-连接hive/tableau连接hive1.jpg)

本文介绍使用tableau连接hive做一些大数据的分析。
<!--more-->

**软件环境**

- mac sierra
- tableau10.3 
- hive(集群使用的是阿里云的E-MapResuce)

## 下载安装ODBC
打开[ODBC官网下载](https://www.cloudera.com/downloads/connectors/hive/odbc/2-5-25.html)链接，选择相应操作系统对应的版本，本文下载的是mac版本的odbc。下载完成后，和一般软件一样点击安装即可。

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-连接hive/tableau连接hive2.jpg)

ps: 需要登录后，方可下载，所以需要注册一个cloudera的账号。

## 连接数据源
### 配置连接信息
打开tableau，依次选择：**连接到服务器**》**更多...**》**Cloudera Hadoop**

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-连接hive/tableau连接hive4.jpg)

- 输入hive的host和端口号(默认为10000)。其中需要确保端口对外开放，没被禁止；
- 类型选择HiveServer2;
- 身份验证选择用户名，默认用户名为cloudera且没有密码。

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-连接hive/tableau连接hive3.jpg)

### 连接数据源
数据源连接信息配置好后，点击登录。如配置没有问题，则会进入以下界面。 
 
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Tableau-连接hive/tableau连接hive5.jpg)

- 第二部分(**架构**)用于选择数据库，直接在输入框中输入数据名。也可直接清空输入框，然后回车，会列出包含的所有数据库；
- 第三部分(表)用于选择表，或编写一些自定义sql。如果清空输入框，回车，会列出已选的数据库下所有的数据表。






