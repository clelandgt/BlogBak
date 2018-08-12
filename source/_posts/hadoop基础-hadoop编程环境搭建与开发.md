---
title: hadoop基础-hadoop编程环境搭建与开发
date: 2018-08-12 01:02:59
tags:
---

开发环境 idea + mac

<!-- MarkdownTOC -->

- 新建项目
    - 新建项目
    - 编译与打包
- wordcount案例

<!-- /MarkdownTOC -->


## 新建项目
### 新建项目

新建项目使用maven的方式并选择java8(可编辑选择已安装的java版本)
![新建项目](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%911.png)


填入Groupid(GroupID是项目组织唯一的标识符，实际对应JAVA的包的结构，是main目录里java的目录结构)和ArtifactID(ArtifactID就是项目的唯一的标识符，实际对应项目的名称，就是项目根目录的名称)
![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%912.png)

设置 `hadoop.version=2.7.2`,编辑pom.xml文件，然后点击`import chage`安装相关依赖。相关依赖会安装在{USER_HOME}/.m2目录

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.insideRia</groupId>
    <artifactId>hdfsFileSystem</artifactId>
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

在【src】->【main】->【java】目录下新建java文件WordCount

![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%913.png)

### 编译与打包
点击File->Project Structure，弹出对话框编辑.

![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%914.png)

选择wordcount
![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%915.png)

只选择需要打包的wordcount,其他hadoop包在集群有，没有必要上传打包。点击apply
![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%916.png)

编译点击【Build】->【Build Artifacts】,生成的java包在{$HOME}/out/artifacts/wordcount_jar/wordcount.jar 

![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%917.png)

wordcount.jar包含以下内容:

![](https://cleland.oss-cn-beijing.aliyuncs.com/blog/img/hadoop%E5%9F%BA%E7%A1%80-hadoop%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E5%BC%80%E5%8F%91/hadoop%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%B8%8E%E7%BC%96%E7%A8%8B%E5%BC%80%E5%8F%918.png)



## wordcount案例

编辑wordcount.txt文件，并上传到hdfs://data/input/wordcount.txt 
 
``` text
hadoop inside hive
walter boy handcome
salary inside hadoop
baby love hadoop
```

编译wordcount打包jar包

``` java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


public class WordCount {
    public static class TokenizerMapper
            extends Mapper<LongWritable, Text, Text, IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException{
            //Hello you
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()){
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable>{
        private IntWritable result = new IntWritable();

        //(Hello, {1, 1})
        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException,InterruptedException{
            int sum = 0;
            for (IntWritable val: values){
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration();
        if(args.length < 2){
            System.err.println("Usage: wordcount <in> [<in>...] <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setReducerClass(IntSumReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```
运行wordcount.jar包

```
$ hadoop jar wordcount.jar /data/input  /data/output
```

## 查询学生总分数
编辑score.txt文件，并上传到并上传到hdfs://data/score/score.txt 

```
1 jack 78 15
2 tome 23 16
3 jane 45 14
1 jack 90 15
2 tome 56 16
3 jane 88 14
```

### 分析

分析map

```
{1, user}, {1, user}
```
分析reduce过程

```
{1, {user, user}}
```

使用mapreduce程序得到学生总分数如下：

```
jack 168 15
tome 79 16
jane 133 14
```

### 源码

UserMapper.java

``` java
package cleland.club;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.StringTokenizer;


public class UserMapper extends Mapper<LongWritable, Text, Text, UserWritable>{

    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException{
        StringTokenizer str = new StringTokenizer(value.toString());
        int counter = 0;

        String id = "";
        String name = "";
        int score = 0;
        short age = 0;
        while(str.hasMoreTokens()){
            if(counter == 0){
                id = str.nextToken();
            }else if(counter == 1){
                name = str.nextToken();
            }else if(counter == 2){
                score = Short.parseShort(str.nextToken());
            }else if(counter == 3){
                age = Short.parseShort(str.nextToken());
            }
            counter += 1;
        }

        context.write(new Text(id), new UserWritable(id,name,score,age));
    }
}
```

UserReduce.java

``` java
package cleland.club;

import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.io.Text;

import java.io.IOException;

public class UserReducer extends Reducer<Text, UserWritable, Text, UserWritable> {

    public void reduce(Text key, Iterable<UserWritable> values, Context context) throws IOException,InterruptedException{
        int totalScore = 0;
        UserWritable resultUser = null;

        for(UserWritable user: values){
            if(resultUser == null){
                resultUser = new UserWritable(user.getId(),user.getName(),user.getScore(),user.getAge());
            }
            totalScore += user.getScore();
        }
        resultUser.setScore(totalScore);
        context.write(key,resultUser);
    }
}
```

UserScoreInfo.java

``` java
package cleland.club;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


public class UserScoreInfo {
    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration();
        if(args.length>2){
            System.err.println("Usage: userscoreinfo <in> [<in>...] <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "student total score");
        job.setJarByClass(UserScoreInfo.class);
        job.setMapperClass(UserMapper.class);
        job.setReducerClass(UserReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(UserWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

UserWritable.java自定义类型

```
package cleland.club;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;


public class UserWritable implements WritableComparable<UserWritable> {
    private String name = "";
    private int score = 0;
    private short age = 0;
    private String id = "";

    //在反序列化时，反射机制需要调用空参构造函数
    public UserWritable(){};

    public UserWritable(String id, String name, int score, short age){
        this.id = id;
        this.name = name;
        this.score = score;
        this.age = age;
    }

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name = name;
    }

    public int getScore(){
        return score;
    }

    public void setScore(int score){
        this.score = score;
    }

    public short getAge(){
        return age;
    }

    public void setAge(short age){
        this.age = age;
    }

    public String getId(){
        return this.id;
    }

    public void setId(String id){
        this.id = id;
    }

    public void set(String id, String name, int score, short age){
        this.id = id;
        this.name = name;
        this.score = score;
        this.age = age;
    }

    @Override
    public int compareTo(UserWritable o){
        int result  = this.name.compareTo(o.getName());
        if (result != 0){
            return result;
        }

        return this.score > o.getScore() ? 1 : -1;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException{
        dataOutput.writeUTF(this.id);
        dataOutput.writeUTF(this.name);
        dataOutput.writeInt(this.score);
        dataOutput.writeShort(this.age);
    }

    @Override
    public void readFields(DataInput dataInput) throws  IOException{
        this.id = dataInput.readUTF();
        this.name = dataInput.readUTF();
        this.score = dataInput.readInt();
        this.age = dataInput.readShort();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        UserWritable that = (UserWritable) o;

        if (score != that.score) return false;
        if (age != that.age) return false;
        if (name != null ? !name.equals(that.name) : that.name != null) return false;
        return id != null ? id.equals(that.id) : that.id == null;
    }

    @Override
    public int hashCode(){
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + score;
        result = 31 * result + (int) age;
        result = 31 * result + (id != null ? id.hashCode() : 0);
        return result;
    }

    @Override
    public String toString(){
        return "name='" + name + '\'' +
                ", score=" + score +
                ", age=" + age +
                ", id='" + id + "'";
    }
}
```




