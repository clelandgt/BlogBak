---
title: Hive-hive基本操作
date: 2018-04-26 00:22:15
tags:
---

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive-hive使用压缩/hive-hive使用压缩1.jpg)
<!-- more -->

<!-- MarkdownTOC -->

- hive简介
    - 优点
    - 缺点
- hive安装
- hive数据类型
    - 基本数据类型
    - 集合数据类型
- 基本操作
    - 创建数据库
        - 新建数据库
        - 查询数据库
        - 修改数据库
        - 删除数据库
    - 表操作
        - 创建表
        - 查看表
        - 删除表
        - 修改表
        - 表的存储格式
    - 数据导入与导出
    - 常用函数
- udf
- 待写
- 参考

<!-- /MarkdownTOC -->




## hive简介
Hive 是基于hadoop的数据仓库，使用该应用程序进行相关的静态数据分析，不需要快速响应出结果，而且数据本身不会频繁变化。

### 优点
1. Hive可以将大多数任务转化为MapReduce的任务.使用SQL查询语句,不需开发原生的Hadoop应用,减少开发量；
2. 内置丰富的函数供调用,如不能满足需求,可开发自定义UDF函数；


### 缺点
1. hive主要是面向OALP的数仓，不适用于OLTP(联机事务处理)；
2. 不支持记录级别的更新、插入、或者删除。

如需实现oltp相关的记录级别的更新、插入、删除可以使用Hbase。

## hive安装
    TODO: 待写

## hive数据类型
### 基本数据类型
|数据类型|长度|例子|
|:--|:--|:--|
|TINYINT|1byte有符号整型|20|
|SMALINT|2byte有符号整型|20|
|INT|4byte有符号整型|20|
|BIGINT|8byte有符号整型|20|
|BOOLEAN|布尔类型(true或false)|TRUE|
|FLOAT|单精度浮点数|3.1415|
|DOUBLE|双精度浮点数|3.1415|
|STRING|字符序列，可使用单/双引号|3.1415|
|TIMESTAMP|时间戳|132788232394(Unix新纪元秒)|
|BINARY|字节数组||

### 集合数据类型
|数据类型|例子|
|:--|:--|
|STRUCT|`STRUCT<name:STRING, age:STRING, sex:STRING>`|
|MAP|`MAP(STRING, FLOAT)`|
|ARRAY|`ARRAY<STRING>`|


## 基本操作
### 创建数据库
Hive创建数据库时，会默认创建相应的目录`hdfs://user/hive/warehouse/test.db`.

#### 新建数据库
``` 
hive> CREATE DATABASE IF NOT EXISTS test;
```

#### 查询数据库
```
hive> SHOW DATABASES;  -- 显示当前所有的数据库
default
test
hive> SHOW DATABASES LIKE 't.*';  -- 使用正则筛选数据库
test
hive> USE test;  -- 进入数据库
hive> SHOW TABLES;  -- 显示所有的数据表
hive> DESCRIBE DATABASE test;  -- 查看数据库信息
test        hdfs://emr-cluster/user/hive/warehouse/test.db
```

#### 修改数据库
可以通过键-值对设置数据库，但是某些元数据信息不可更改，如：数据库名和数据库所在的目录位置。

```
hive> ALTER DATABASE test SET DBPROPERTIES('edited-by'='Cleland');
```

#### 删除数据库
```
hive> DROP DATABASE IF EXISTS test;  -- 删除数据库(数据库下无数据表，如有会执行失败)
hive> DROP DATABASE IF EXISTS test CASCADE;  -- 强制删除数据库(先清表再删除数据库)
```


### 表操作
#### 创建表
内部表vs外部表

1. 内部表drop时会删除数据源，而外部表不会；
2. 内部表不方便与外部表共享数据，而外部表可以。

目前使用阿里的**EMR+OSS**计算与存储分离，所以一些关键的表都使用的是外部表。

**内部表**

``` sql
CREATE TABLE IF NOT EXISTS test.employees( 
    name  STRING COMMENT 'Employee name',
    salary  FLOAT COMMENT 'Employee salary',
    subordinates  ARRAY<STRING> COMMENT 'Names of subordinates',
    deductions  MAP<STRING, FLOAT> COMMENT 'Keys are deductions names, values are percentages',
    address  STRUNCT<street:STRING, city:STIRNG, state:STRING, zip:INT> COMMENT 'Home address'
COMMENT 'Description of the table'
TBLPROPERTIES('creator'='me', 'created_at'='2012-01-02 10:00:00', ...)
LOCATION '/user/hive/warehouse/test.db/employees';
)
```

**外部表**

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS test.stocks(
    exchange  STRING,
    symbol  STRING,
    ymd  STRING,
    price_open  FLOAT,
    price_high  FLOAT,
    price_low  FLOAT,
    price_close  FLOAT,
    volume  FLOAT,
    price_adj_close FLOAT,
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/data/stocks'
```

#### 查看表
``` 
hive> USE test;
hive> SHOW TABLES;
employees
stocks
hive> SHOW TABLES IN test;
hive> DESCRIBE EXTENDED test.employees;  -- 查看详细表结果
hive> DESCRIBE FORMATTED test.employees;  -- 查看详细表结果，比EXTENDED内容更多。
``` 

#### 删除表
```
hive> DROP TABLE IF EXISTS test.employees;
```
由于hdfs的回收机制，数据会移动到`/user/$USER/.Trash`

#### 修改表
**表属性**

```
hive> ALTER TABLE table1 RENAME TO table2;  -- 表table1重名为table2
hive> ALTER TABLE log_messages SET TBLPROPERTIES('notes'='The process id is no longer captured')  -- 修改表属性
hive> ALTER TABLE log_messages PARTITION(year=2012,month=1,day=1) SET FILEFORMAT SEQUENCFILE;  -- 修改存储属性
```

**列**

```
hive> ALTER TABLE log_messages CHANGE COLUMN hms hours_minutes_seconds INT COMMENT 'The houes, minutes, and seconds part of the timestamp AFTER severity';  -- 修改列
hive> ALTER TABLE log_messages ADD COLUMNS(app_name STRING, session_id STRING);  -- 增加列
hive> -- 待补充删除或者替换列
```

**分区**

```
hive> ALTER TABLE log_messages ADD IF NOT EXISTS PATITION(year=2011,month=1,day=1) LOCATION '/log/2011/01/01';  -- 添加分区
hive> ALTER TABLE log_messages PARTITION(year=2011,month=12,day=2) SET LOCATION 's3n://ourbucket/logs/2011/01/01'  -- 修改分区(不会移动或删除数据)
hive> ALTER TABLE log_messages DROP IF EXISTS PARTITION(year=2011,month=12,day=2);  -- 删除分区
```


#### 表的存储格式
将在高级用法中补充**存储格式+压缩+分区+分桶**实现hive的优化。


### 数据导入与导出



### 常用函数


## udf

## 待写

- 性能优化1 (配置与sql相关优化)
- 性能优化2（分表+分桶+压缩 [参考连接](https://blog.csdn.net/djd1234567/article/details/51581312)）



## 参考
- 《Hive编程指南》
- [Hive官方教程](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-GettingStarted)


