---
title: Hive 基本操作
date: 2018-03-11 18:22:09
tags: hive
---

![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive-hive使用压缩/hive-hive使用压缩1.jpg)
<!-- more -->

<!-- MarkdownTOC -->

- hive简介
    - 优点
    - 缺点
    - hadoop2.x生态
    - hive架构图
- hive安装
- hive数据类型
    - 基本数据类型
    - 集合数据类型
- 基本操作
    - 数据库操作
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
    - 装载数据
- 高级主题
- 参考

<!-- /MarkdownTOC -->

## hive简介
Hive 是基于hadoop的数据仓库，使用该应用程序进行相关的静态数据分析，不需要快速响应出结果，而且数据本身不会频繁变化。

### 优点
1. Hive可以将大多数任务转化为MapReduce的任务.使用SQL查询语句,不需开发原生的Hadoop应用,减少开发量；
2. 内置丰富的函数供调用,如不能满足需求,可开发自定义UDF函数；


### 缺点
1. hive主要是面向OLAP的数仓，不适用于OLTP(联机事务处理)；
2. 不支持记录级别的更新、插入、或者删除。

如需实现oltp相关的记录级别的更新、插入、删除可以使用Hbase。

### hadoop2.x生态
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive%E5%9F%BA%E7%A1%80-%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/2.png)

### hive架构图
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive%E5%9F%BA%E7%A1%80-%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/1.png)


## hive安装
    TODO: 待写

## hive数据类型
### 基本数据类型
|数据类型|长度|数据范围|例子|
|:--|:--|:--|:--|
|TINYINT|1byte有符号整型|-128~127(-2^7~2^7-1)|20|
|SMALINT|2byte有符号整型|-32,768~32,767(-2^15~2^15-1)|20|
|INT|4byte有符号整型|-2,147,483,648~2,147,483,647(-2^31~2^31-1)|20|
|BIGINT|8byte有符号整型|-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807(-2^63~2^63-1)|20|
|BOOLEAN|布尔类型(true或false)||TRUE|
|FLOAT|4byte单精度浮点数||3.1415|
|DOUBLE|8byte双精度浮点数||3.1415|
|STRING|字符序列，可使用单/双引号||3.1415|
|TIMESTAMP|时间戳||132788232394(Unix新纪元秒)|
|BINARY|字节数组|||


### 集合数据类型
|数据类型|例子|
|:--|:--|
|STRUCT|`STRUCT<name:STRING, age:STRING, sex:STRING>`|
|MAP|`MAP(STRING, FLOAT)`|
|ARRAY|`ARRAY<STRING>`|


## 基本操作
### 数据库操作
#### 创建数据库
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
参考Hive官方文档 [LanguageManual DDL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)

#### 创建表
内部表vs外部表

1. 内部表drop时会删除数据源，而外部表不会；
2. 内部表不方便与其他组件框架(spark等)共享数据，而外部表可以。

目前使用阿里的**EMR+OSS**计算与存储分离，所以一些关键的表都使用的是外部表。

**内部表**

``` sql
-- 建表
DROP TABLE IF EXISTS test.employees;
CREATE TABLE IF NOT EXISTS test.employees( 
    emp_no INT COMMENT '员工ID',
    birth_date STRING COMMENT '出生日期',
    first_name STRING COMMENT '名',
    last_name STRING COMMENT '姓',
    gender STRING COMMENT '性别',
    hire_date STRING COMMENT '入职时间',
    title STRING COMMENT '职称'
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
TBLPROPERTIES ("skip.header.line.count"="1");

-- 加载数据
LOAD DATA INPATH '/data/employees.csv' OVERWRITE INTO TABLE test.employees;

-- 查看数据
SELECT * FROM test.employees
```

**PS：**

- 使用load方法加载数据时，将文件employees.csv移动到 /user/hive/warehouse/test.db/employees.csv. 会删除源文件；
- 如果是文件没在hdfs上，在本地。则在`INPATH`前加上`LOCAL`


**外部表**

``` sql
-- 建表
DROP TABLE IF EXISTS test.employees_ext;
CREATE EXTERNAL TABLE IF NOT EXISTS test.employees_ext( 
    emp_no INT COMMENT '员工ID',
    birth_date STRING COMMENT '出生日期',
    first_name STRING COMMENT '名',
    last_name STRING COMMENT '姓',
    gender STRING COMMENT '性别',
    hire_date STRING COMMENT '入职时间'
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
LOCATION '/data/employees_dir'
TBLPROPERTIES ("skip.header.line.count"="1");

-- 查看数据
SELECT * FROM test.employees_ext 
```

#### 查看表
``` 
-- 查看表
hive> USE test;
hive> SHOW TABLES; 
hive> SHOW TABLES IN test;
hive> SHOW TABLES IN test LIKE 'employees'

-- 查看表的属性
hive> SHOW CREATE TABLE employees;  -- 查看建表语句
hive> DESCRIBE EXTENDED test.employees;  -- 查看详细表结果
hive> DESCRIBE FORMATTED test.employees;  -- 查看详细表结果，比EXTENDED内容更多。
``` 

#### 删除表
```
hive> TRUNCATE test.employees;  -- 清空表数据
hive> DROP TABLE IF EXISTS test.employees;  -- 删除表
```
由于hdfs设置了回收机制，数据会移动到`/user/$USER/.Trash`，所以drop表了不用怕，可从回收站中恢复(只增对表数据放在hdfs的情况)。

#### 修改表

**表属性**

```
hive> ALTER TABLE test.employees RENAME TO test.employees_2;  -- 表table1重名为table2
hive> ALTER TABLE test.employees SET TBLPROPERTIES ('comment'='employees info')  -- 修改表属性
hive> ALTER TABLE test.employees SET FILEFORMAT SEQUENCFILE;  -- 修改存储属性
```

**分区(重要)**

```
hive> SHOW PARTITIONS test.employees_part;  -- 查看分区
hive> ALTER TABLE test.employees_part DROP PARTITION(gender='F');
hive> ALTER TABLE test.employees_part ADD PARTITION(gender='F');
```

### 装载数据
1. 直接将数据放入表指定或默认的位置(内部表：/user/hive/warehouse)
2. 使用load

    ```
    hive> LOAD DATA INPATH '/data/employees.csv' OVERWRITE INTO TABLE test.employees;  -- hdfs
    hive> LOAD DATA LOCAL INPATH '/data/employees.csv' OVERWRITE INTO TABLE test.employees;  -- 本地
    ```
3. 分区表，指定分区路径

   见上文【表操作】->【修改表】-> 【分区】
    
4. 将查询数据插入表

    ```
    CREATE TABLE test.employees_10000 LIKE test.employees;
    INSERT OVERWRITE TABLE test.employees_10000
    SELECT 
        *
    FROM test.employees
    LIMIT 10000
    ```
5. 快速复制表
    
    **非分区复制**
    
    ```
    create table t_copy as select * from t_temp
    ```
    
    **分区复制**
    
    ```
    create table t_copy like t_part;  -- 复制表结果
    dfs -cp /user/hive/warehouse/test.db/t_part/* /user/hive/warehouse/test.db/t_copy/;  -- 复制表数据
    msck repair table t_copy;  -- 修复分区


### 常用函数

``` 
--  聚合函数
count(expr)
sum(col)
avg(col)
min(col)
max(col)
collect_set(col)  -- 返回集合col元素排重后的数组(select非groupby的字段时可使用)

-- 日期函数
to_date(STRING timestamp)  -- 返回时间字符串的日期部分，例如：to_date('1970-01-01 00:00:00')='1970-01-01'
datediff(STRING start_date, STRING end_date)  -- 计算两个日期之间的天数，例如：datediff('2018-04-04', '2018-04-01')=3
date_add(STRING start_date, INT day)  -- 从日期减去day天数，例如：date_add('2018-04-01', 4)='2018-04-05'
date_sub(STRING start_date, INT day)  -- 从日期减去day天数，例如：date_sub('2018-04-20', 4)='2018-04-16'

year(STRING date)  -- 返回时间字符串中的年
month(STRING date)  -- 返回时间字符串中的月
day(STRING date)  -- 返回时间字符串中的日期
hour(STRING date)  -- 返回时间字符串中的时
minute(STRING date)  -- 返回时间字符串中的分
second(STRING date)  -- 返回时间字符串中的秒
weekofyear(STRING date)  -- 一年的第几周

--  其他内置
concat(STRING s1, STRING s2)
concat_ws(STRING separator, STRING s1, STRING s2)
format_number(NUMBER x, INT d)
get_json_object(STRING json_string, STRING PATH)
length(STRING s)
ltrim(STRING s)  -- 将字符串s前面出现的空格全部去除掉
rtrim(STRING s)  -- 将字符串s后面出现的空格全部去除掉
trim(STRING s)  -- 将字符串s前后面出现的空格全部去除掉
```

## 查询
- 不支持子查询
- group by
- 关联查询

### 不支持子查询
hive不支持子查询，需要将子查询转化为外层join的形式。

### Group By

```
SELECT 
    ti.emp_no
    ,count(1) as num
FROM test.employees em
INNER JOIN test.title ti on ti.emp_no=em.emp_no
GROUP BY ti.emp_no
HAVING num>1
ORDER BY num DESC
```

ps: select选取的非数值字段维度只能是group的维度，不能写入其他维度。

### 关联查询
- join(等同于inner join 内连接)
- left outer join(等同于 left join 左外连接)
- right outer join(等同于 right join 右外连接)
- full outer join(全连接)
- left semi join
- 笛卡尔积 Join

## 分区与分桶
TODO: 优点与注意点

```
-- 建立表
DROP TABLE IF EXISTS test.employees_part;
CREATE TABLE  IF NOT EXISTS test.employees_part(
    emp_no INT COMMENT '员工ID',
    birth_date STRING COMMENT '出生日期',
    first_name STRING COMMENT '名',
    last_name STRING COMMENT '姓',
    hire_date STRING COMMENT '入职时间'
)
PARTITIONED BY(gender STRING)
CLUSTERED BY (emp_no) SORTED BY(emp_no) INTO 10 BUCKETS

-- 插入数据
SET hive.exec.dynamic.partition=true;
-- 严格型必需包括至少一个静态分区  
SET hive.exec.dynamic.partition.mode=nostrict;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
INSERT OVERWRITE TABLE test.employees_part PARTITION (gender)
SELECT 
    emp_no
    ,birth_date
    ,first_name
    ,last_name
    ,hire_date
    ,gender
FROM test.employees
```

## 压缩
hive中的数据使用压缩的好处(执行查询时会自动解压)：

1. 可以节约磁盘的空间，基于文本的压缩率可达40%+;
2. 压缩可以增加吞吐量和性能量(减小载入内存的数据量)，但是在压缩和解压过程中会增加CPU的开销。所以针对IO密集型的jobs(非计算密集型)可以使用压缩的方式提高性能。

### 主流的压缩算法
查看集群的支持的压缩算法.

``` shell
$ hive -e "set io.compression.codecs"
io.compression.codecs=org.apache.hadoop.io.compress.DefaultCodec,
org.apache.hadoop.io.compress.GzipCodec,
org.apache.hadoop.io.compress.BZip2Codec,
org.apache.hadoop.io.compress.DeflateCodec,
org.apache.hadoop.io.compress.SnappyCodec,
org.apache.hadoop.io.compress.Lz4Codec
```

常用的压缩算法：

- Snappy
- Gzip
- ZLIB


### hive文件格式
- **TEXTFILE**: 默认格式，数据不做压缩，磁盘开销大。如需压缩，可使用Gzip,Bzip2压缩算法，但是不会对数据进行切分;
- **SEQUENCEFILE**: 二进制文件，具有使用方便、可分割、可压缩.SequenceFile支持三种压缩选择：NONE，RECORD(压缩率低)，BLOCK(常用且压缩性能最好);
- **RCFILE**: RCFILE是一种行列存储相结合的存储方式;
- **ORCFILE**: 0.11以后出现.

### hive配置压缩
建表时申明文件的存储格式，默认为TEXTFILE。如使用压缩常使用分块压缩**SEQUENCEFILE**。

```
CREATE TABLE A(
    ...
)
STORED AS SEQUENCEFILE
```

数据处理的中间过程和结果使用**Snappy**算法进行压缩。

```
-- 任务中间压缩
set hive.exec.compress.intermediate=true;
set hive.intermediate.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
set hive.intermediate.compression.type=BLOCK;

-- map/reduce 输出压缩
set hive.exec.compress.output=true;
set mapred.output.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapred.output.compression.type=BLOCK;
```

### 压缩测试案例
Hive原始数据为119.2G。

| 压缩算法 | TEXTFILE格式 | SEQUENCEFILE | RCFILE | ORCFILE |
| :-- | :-- | :-- | :-- | :-- |
| 不压缩  | 119.2G | 54.1G | 20.0G | 98G | 
| Snappy | 30.2G | 23.6G | 13.6G | 27.0G |
| Gzip | 18.8G | 14.1G | 不支持 | 15.2 G | 
| ZLIB | 不支持 | 不支持 | 10.1G | 不支持 |

## 与外部数据的集成
### mongo
```
DROP TABLE OLTP.house_mogu;
CREATE EXTERNAL TABLE OLTP.house_mogu(
    id STRING,
    platform_config ARRAY<MAP<STRING, STRING>>,
    publish_time STRING,
    house_id STRING,
    room_info STRING,
    area_name STRING,
    broker_info MAP<STRING, STRING>
)
PARTITIONED BY(batch_num STRING)
ROW FORMAT SERDE 'com.mongodb.hadoop.hive.BSONSerDe'
WITH SERDEPROPERTIES ('mongo.columns.mapping'='{
        "id":"_id",
        "platform_config":"platform_config",
        "publish_time":"publish_time",
        "house_id":"house_id",
        "room_info":"room_info",
        "area_name":"area_name",
        "broker_info":"broker_info"
    }'
)
STORED AS INPUTFORMAT 'com.mongodb.hadoop.mapred.BSONFileInputFormat'
OUTPUTFORMAT 'com.mongodb.hadoop.hive.output.HiveBSONFileOutputFormat'
LOCATION 'oss://${BIGDATA_HOME}/hive/warehouse/oltp.db/house_mogu';
```


### hbase
需要在hbase建立house_width表

```
CREATE EXTERNAL TABLE OLAP.house_width(
    house_id STRING,
    src_type INT,
    crawl_time STRING,
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,
    HSInfo:house_id,
    HSInfo:src_type,
    HSInfo:crawl_time
")
TBLPROPERTIES ("hbase.table.name" = "house_width");

```

## 配置hive的几种方式
1. 修改${HIVE_HOME}/conf/hive-site.xml配置文件（永久有效)
    
    ```
    <property>
        <name>mapreduce.job.queuename</name>
        <value>batch</value>
    </property>
    ```

2. 命令行参数（当次有效）

    ```
    $ hive --hiveconf mapreduce.job.queuename=batch
    ```

3. 在已经进入cli时进行参数声明（当次有效）。
    
    ```
    hive> SET mapreduce.job.queuename=batch
    ```

## 使用python第三方接口
首先安装第三方库

```
$ pip install pyhs2
```

测试代码

``` python
import pyhs2

with pyhs2.connect(host='47.94.36.22',
                   port=10000,
                   authMechanism="PLAIN",
                   user='root',
                   password='xxxx',
                   database='test') as conn:
    with conn.cursor() as cur:
        #Show databases
        print cur.getDatabases()

        #Execute query
        cur.execute("select * from title limit 10")

        #Return column info from query
        print cur.getSchema()

        #Fetch table results
        for i in cur.fetch():
            print i

```

## 高级主题
- hive核心流程分析
- 执行计划
- 常用的优化手段
- 数据倾斜
- udf
- 权限管理
- 锁管理

## 参考
- 《Hive编程指南》
- [Hive官方教程](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-GettingStarted)
- [mysql官方开源数据集说明](https://dev.mysql.com/doc/employee/en/)

