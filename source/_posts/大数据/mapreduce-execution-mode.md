---
title: MapReduce程序运行模式
cover: false
top: false
date: 2020-08-03 14:09:07
group: bigdata
permalink: mapreduce-execution-mode
categories: 大数据
tags:
- MapReduce
- 大数据
keywords:
- MapReduce程序运行模式
summary: 本文详细介绍了几种MapReduce程序运行模式
---

MR程序运行方式



1、在集群上运行

使用maven打包jar，拷贝jar包到集群上，运行`hadoop jar`或`yarn jar`命令执行。

如：

```shell
$ hadoop jar hadoop-test-0.0.1-SNAPSHOT.jar com.yglong.hadoop.mapred.wordcount.WordCount /data/wc/input /data/wc/output
```

```shell
$ yarn jar hadoop-test-0.0.1-SNAPSHOT.jar com.yglong.hadoop.mapred.wordcount.WordCount /data/wc/input /data/wc/output
```

运行后查看结果：

```shell
$ hadoop fs -ls /data/wc/output
Found 2 items
-rw-r--r--   2 bigdata supergroup          0 2020-08-03 05:25 /data/wc/output/_SUCCESS
-rw-r--r--   2 bigdata supergroup         36 2020-08-03 05:25 /data/wc/output/part-r-00000
```

2、在windows本地运行

在windows上可以有两种运行方式：

- 在windows本地运行
- 从windows上提交到集群上运行

2.1 在windows本地运行

在windows本地运行MR程序，主要方便在开发Map和Reduce方法时调试代码，因为可以直接在本地如IDEA上运行程序，验证程序的正确性，也可以在IDEA中debug程序的运行情况。

2.1.1 安装并配置本地hadoop环境

在windows上安装hadoop，与在linux上类似，从官网下载hadoop压缩包，解压到某个本地目录。

设置HADOOP_HOME，JAVA_HOME环境变量

添加Path环境变量：

```bash
%HADOOP_HOME%\bin;%JAVA_HOME%\bin;
```

注意，在windows上需要对hadoop做一个特殊的配置，下载winutils并安装到`%HADOOP_HOME%\bin`目录下。

如果不配置这一步，当运行MR程序时，会遇到类似如下的报错：

```java
2020-08-03 13:59:43,264 [myid:] - WARN  [main:Shell@693] - Did not find winutils.exe: {}
java.io.FileNotFoundException: Could not locate Hadoop executable: C:\ylong\tool\hadoop-3.2.1\bin\winutils.exe -see https://wiki.apache.org/hadoop/WindowsProblems
	at org.apache.hadoop.util.Shell.getQualifiedBinInner(Shell.java:619)
	at org.apache.hadoop.util.Shell.getQualifiedBin(Shell.java:592)
	at org.apache.hadoop.util.Shell.<clinit>(Shell.java:689)
	at org.apache.hadoop.util.GenericOptionsParser.preProcessForWindows(GenericOptionsParser.java:520)
	at org.apache.hadoop.util.GenericOptionsParser.parseGeneralOptions(GenericOptionsParser.java:571)
	at org.apache.hadoop.util.GenericOptionsParser.<init>(GenericOptionsParser.java:174)
	at org.apache.hadoop.util.GenericOptionsParser.<init>(GenericOptionsParser.java:156)
	at com.yglong.hadoop.mapred.wordcount.WordCount.main(WordCount.java:49)
```

[winutils下载地址](https://github.com/cdarlint/winutils)

需要下载winutils的版本必须与安装的hadoop版本一样，如我本地安装的hadoop 3.2.1，所以需要下载[hadoop 3.2.1对应的winutils](https://github.com/cdarlint/winutils/tree/master/hadoop-3.2.1/bin)

下载所有文件（包括winutils.exe, hadoop.dll等）后，全部拷贝到`%HADOOP_HOME%\bin`目录下。

2.1.2 代码设置

本地hadoop环境配置好后，就可以在本地开发MR程序了。

首先，需要引入hadoop依赖包，在pom.xml中添加：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <hadoopVersion>3.2.1</hadoopVersion>
</properties>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>${hadoopVersion}</version>
</dependency>
```

日志配置

如果要在IDEA中看到MR执行日志，需要配置日志。

pom.xml中添加SLF的实现依赖，如：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.25</version>
</dependency>
```

然后在src/main/resources下添加log4j.properties文件，如：

```properties
hadoop.root.logger=INFO, CONSOLE
hadoop.console.threshold=INFO
log4j.rootLogger=${hadoop.root.logger}

log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.Threshold=${hadoop.console.threshold}
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-5p [%C]: %m%n
```

就这样，开发好MR程序后，即可直接Run起来了。

代码中不做任何设置，默认情况下是在local运行的，且输入输出文件目录为本地文件目录，而不是HDFS目录，所以在代码中设置输入输出文件路径时，需要指定windows本地路径。如：

```java
Path infile = new Path("C:\\tmp\\data\\wc\\input");
FileInputFormat.addInputPath(job, infile);

Path outfile = new Path("C:\\tmp\\data\\wc\\output");
if (outfile.getFileSystem(conf).exists(outfile)) {
    outfile.getFileSystem(conf).delete(outfile, true);
}
FileOutputFormat.setOutputPath(job, outfile);
```

为了不在程序中写死路径，也可以通过参数提供输入输出文件路径，这样就可以在代码中通过参数获取：

使用GenericOptionsParser工具类：

```java
String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
Path infile = new Path(otherArgs[0]);
FileInputFormat.addInputPath(job, infile);

Path outfile = new Path(otherArgs[1]);
if (outfile.getFileSystem(conf).exists(outfile)) {
    outfile.getFileSystem(conf).delete(outfile, true);
}
FileOutputFormat.setOutputPath(job, outfile);
```

在程序参数上提供本地路径。



运行后，可以在本地目录下生成结果文件。





如果为了在本地提交到远程集群运行MR程序而在classpath中如src/main/resources中添加了集群的配置文件（core-site.xml, hdfs-site.xml, yarn-site.xml, mapred-site.xml）

这种情况下，会优先读取配置文件的信息，从而会尝试提交到远程集群运行。会报类似如下错误：

```
Exception in thread "main" java.lang.IllegalArgumentException: Pathname /C:/tmp/data/wc/output from C:/tmp/data/wc/output is not a valid DFS filename.
	at org.apache.hadoop.hdfs.DistributedFileSystem.getPathName(DistributedFileSystem.java:236)
	at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1576)
	at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1573)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1588)
	at org.apache.hadoop.fs.FileSystem.exists(FileSystem.java:1683)
	at com.yglong.hadoop.mapred.wordcount.WordCount.main(WordCount.java:89)
```



为了强制在本地运行，可以设置两个配置项：

```java
Configuration conf = new Configuration();
// 如果在windows本地执行MR程序，必须将mapreduce.framework.name设置为local
conf.set("mapreduce.framework.name", "local");
// 有些版本也需要将fs.defaultFS设置为file:///，让MR程序查找本地路径，而不是HDFS路径。
// 经测试，hadoop-3.2.1必须设置
conf.set("fs.defaultFS", "file:///");
```

当然为了不在程序中写死，也可以在通过命令参数`-D`提供：

```java
-Dmapreduce.framework.name=local -Dfs.defaultFS=file:///
```



从windows上提交到集群上运行

在windows本地运行时，默认时在本地hadoop中运行的。如果在windows本地提交到远程的集群中运行，需要先将集群的配置文件添加到本地classpath中，如添加到src/main/resources目录下。

配置文件包括：core-site.xml, hdfs-site.xml, yarn-site.xml, mapred-site.xml



问题1：

```java
2020-08-03 15:02:40,613 INFO  [org.apache.hadoop.mapreduce.Job]: Job job_1596433378472_0001 failed with state FAILED due to: Application application_1596433378472_0001 failed 2 times (global limit =5; local limit is =2) due to AM Container for appattempt_1596433378472_0001_000002 exited with  exitCode: 1
Failing this attempt.Diagnostics: [2020-08-03 07:02:37.497]Exception from container-launch.
Container id: container_1596433378472_0001_02_000001
Exit code: 1

[2020-08-03 07:02:37.561]Container exited with a non-zero exit code 1. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
/bin/bash: line 0: fg: no job control

[2020-08-03 07:02:37.561]Container exited with a non-zero exit code 1. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
/bin/bash: line 0: fg: no job control

For more detailed output, check the application tracking page: http://bigdata02:8088/cluster/app/application_1596433378472_0001 Then click on links to logs of each attempt.
. Failing the application.
2020-08-03 15:02:40,630 INFO  [org.apache.hadoop.mapreduce.Job]: Counters: 0
```

如果遇到如上的类似`/bin/bash`之类的错误，那就是忘记了配置支持跨平台的参数：

```java
// 如果从windows本地提交MR程序到集群运行，必须设置mapreduce.app-submission.cross-platform为true，支持跨平台
conf.set("mapreduce.app-submission.cross-platform", "true");
```

这个参数默认值是false。



问题2：

```java
2020-08-03 15:21:09,920 INFO  [org.apache.hadoop.mapreduce.JobSubmitter]: Cleaning up the staging area /opt/software/hadoop/yarn/staging/ylong/.staging/job_1596438997159_0003
Exception in thread "main" org.apache.hadoop.security.AccessControlException: Permission denied: user=ylong, access=EXECUTE, inode="/opt":bigdata:supergroup:drwx------
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.check(FSPermissionChecker.java:399)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkTraverse(FSPermissionChecker.java:315)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:242)
	at org.apache.hadoop.hdfs.server.namenode.FSPermissionChecker.checkPermission(FSPermissionChecker.java:193)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1879)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkPermission(FSDirectory.java:1863)
	at org.apache.hadoop.hdfs.server.namenode.FSDirectory.checkOwner(FSDirectory.java:1808)
	at org.apache.hadoop.hdfs.server.namenode.FSDirAttrOp.setPermission(FSDirAttrOp.java:63)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.setPermission(FSNamesystem.java:1905)
	at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.setPermission(NameNodeRpcServer.java:876)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.setPermission(ClientNamenodeProtocolServerSideTranslatorPB.java:533)
	at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:528)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1070)
	at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:999)
	at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:927)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1730)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2915)
```

HDFS目录权限问题，暴力点直接把报错的目录设置为777，如上报错的目录是/opt，所以，将/opt和子目录全部设置为777：

```shell
$ hadoop fs -chmod -R 777 /opt
```

问题3：

```java
Error: java.lang.RuntimeException: java.lang.ClassNotFoundException: Class com.yglong.hadoop.mapred.wordcount.WordCount$TokenizerMapper not found
	at org.apache.hadoop.conf.Configuration.getClass(Configuration.java:2638)
	at org.apache.hadoop.mapreduce.task.JobContextImpl.getMapperClass(JobContextImpl.java:187)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:759)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:347)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:174)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1730)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:168)
```

这个错很明显了，运行时找不到自定义的Mapper类。

在本地远程提交集群运行时，必须设置要提交的程序jar包，如下：

```java
job.setJar("C:\\ylong\\workspace\\study\\hadoop\\target\\hadoop-test-0.0.1-SNAPSHOT.jar");
```

这个jar包需要在本地先打出来，如使用maven打包。

其实就是将我们自己开发的Mapper和Reducer类打包到jar里，它们需要被提交到集群中并运行。



总结，在本地提交集群运行MR程序，需要做的事情：

- 拷贝集群配置文件到classpath
- 设置mapreduce.app-submission.cross-platform为true
- 将自定义类打成jar包，并设置到job里
- 如果遇到HDFS目录权限问题，设置目录权限