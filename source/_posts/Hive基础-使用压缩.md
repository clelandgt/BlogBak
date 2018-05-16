---
title: Hive|hive使用压缩
date: 2018-03-11 18:22:09
tags: hive
---
![](http://cleland.oss-cn-beijing.aliyuncs.com/blog/img/Hive-hive使用压缩/hive-hive使用压缩1.jpg)
hive中的数据使用压缩的好处(执行查询时会自动解压)：

1. 可以节约磁盘的空间，基于文本的压缩率可达40%+;
2. 压缩可以增加吞吐量和性能量(减小载入内存的数据量)，但是在压缩和解压过程中会增加CPU的开销。所以针对IO密集型的jobs(非计算密集型)可以使用压缩的方式提高性能。

<!-- more -->

## 主流的压缩算法
查看集群的支持的压缩算法.

``` shell
hive -e "set io.compression.codecs"
```

返回支持的压缩算法
``` shell
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


## hive文件格式
- **TEXTFILE**: 默认格式，数据不做压缩，磁盘开销大。如需压缩，可使用Gzip,Bzip2压缩算法，但是不会对数据进行切分;
- **SEQUENCEFILE**: 二进制文件，具有使用方便、可分割、可压缩.SequenceFile支持三种压缩选择：NONE，RECORD(压缩率低)，BLOCK(常用且压缩性能最好);
- **RCFILE**: RCFILE是一种行列存储相结合的存储方式;
- **ORCFILE**: 0.11以后出现.


## hive配置压缩
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

## 压缩测试案例
Hive原始数据为119.2G。

| 压缩算法 | TEXTFILE格式 | SEQUENCEFILE | RCFILE | ORCFILE |
| :-- | :-- | :-- | :-- | :-- |
| 不压缩  | 119.2G | 54.1G | 20.0G | 98G | 
| Snappy | 30.2G | 23.6G | 13.6G | 27.0G |
| Gzip | 18.8G | 14.1G | 不支持 | 15.2 G | 
| ZLIB | 不支持 | 不支持 | 10.1G | 不支持 |


## 参考
- 《Hive编程指南》
- http://blog.csdn.net/hereiskxm/article/details/42171325
