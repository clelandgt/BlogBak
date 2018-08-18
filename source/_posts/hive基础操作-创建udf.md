---
title: hive基础操作-创建udf
date: 2018-08-18 16:47:51
tags:
---

开发环境：idea+maven

<!-- MarkdownTOC -->

- maven配置文件
- 编写java程序
- 创建模板函数并使用UDF

<!-- /MarkdownTOC -->


## maven配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cleland.club</groupId>
    <artifactId>my_udf</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <hadoop.version>2.7.2</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
        <dependency>
            <groupId>org.apache.mrunit</groupId>
            <artifactId>mrunit</artifactId>
            <version>1.1.0</version>
            <classifier>hadoop2</classifier>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-minicluster</artifactId>
            <version>${hadoop.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>2.0.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>2.0.1</version>
        </dependency>

    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

编写udf需添加hive依赖

```
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.0.1</version>
</dependency>

<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>2.0.1</version>
</dependency>
```

## 编写java程序
编写java程序完成自定义udf: 将小写转化为大写。

```
package my_udf;

import org.apache.hadoop.hive.ql.exec.UDF;


public class MyUpper extends UDF{
    public String evaluate(String intput){
        return intput.toUpperCase();
    }

    public static void main(String[] args){
        MyUpper up = new MyUpper();
        System.out.println(up.evaluate(new String("test")));
    }
}

```

打jar包并上传到hdfs的/lib/my_udf.jar。

## 创建模板函数并使用UDF

```
hive> add jar hdfs:/lib/my_udf.jar;
hive> CREATE TEMPORARY FUNCTION MyUpper AS 'my_udf.MyUpper';
hive> select MyUpper("input")
INPUT
```

