---
title: 安装Zookeeper集群
cover: false
top: false
date: 2020-07-07 14:09:07
group: bigdata
permalink: install-zookeeper-cluster
categories: 大数据
tags:
- BigData
- 大数据
keywords:
- Zookeeper集群搭建
- 安装Zookeeper集群
summary: 本文详细介绍了如何搭建Zookeeper集群
---

### 一、前言

`Zookeeper`的用途非常广泛，它是分布式协调的最常用组件。

对于学习大数据，最基本的是学习`Hadoop`，而搭建高可用的`Hadoop`集群，必然需要首先搭建`Zookeeper`集群。本文将详细介绍如何搭建一个`Zookeeper`集群，为搭建`Hadoop`集群做好基础准备。

### 二、环境准备

本文将一步一步搭建一个具有3个节点的`Zookeeper`集群，所有节点已安装`Ubuntu 18.04`操作系统。

在安装`Zookeeper`之前，需要做一些环境准备。

#### 1. 设置hostname

首先需要设置3个节点的`hostname`，比如分别为：`bigdata01`, `bigdata02`, `bigdata03`
```shell
vim /etc/hostname
```
分别登录每个节点，在`/etc/hostname`中修改hostname为实际的hostname，如`bigdata01`

#### 2. 创建用户

一般情况下，最好创建一个专用的用户来运行`Zookeeper`，比如：`bigdata`。

> 你也可以跳过这一步，直接用`root`或已有用户来运行`Zookeeper`。

以`root`用户分别登录每个节点，执行命令：

```shell
useradd -m bigdata -d /home/bigdata -s /bin/bash
```

然后设置新用户密码：
```shell
passwd bigdata
```

#### 3. 设置免密登录

接下来，需要配置各个节点之间可以以新创建的用户`免密登录`，即可以通过`ssh key`的方式彼此登录。

1）首先，以新创建的用户（如：`bigdata`）登录到第一个节点（如：`bigdata01`），执行`ssh-keygen`生成`ssh key`：

```shell
ssh-keygen
```

2）然后进入`/home/bigdata/.ssh`目录下，执行`ssh-copy-id`命令将`ssh key`分别拷贝到3个节点上：

```shell
cd ~/.ssh

ssh-copy-id -i id_rsa.pub bigdata01
ssh-copy-id -i id_rsa.pub bigdata02
ssh-copy-id -i id_rsa.pub bigdata03
```

3）分别登录另外两个节点，重复以上步骤1）和2）。

#### 4. 安装JDK并配置JAVA_HOME

如果系统中没有安装`JDK`，需要先安装。

如下命令安装`OpenJDK 8`：

```shell
apt-get update
apt-get install openjdk-8-jdk-headless
```

然后配置`JAVA_HOME`:

```shell
vim /etc/profile
```

在`/etc/profile`最后加上如下两行：
```java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

保存退出，执行如下命令使`JAVA_HOME`环境变量生效：
```shell
source /etc/profile
```

> 所有节点都需要执行以上步骤安装`JDK`，以及配置`JAVA_HOME`环境变量。

### 三、安装Zookeeper集群

#### 1. 下载Zookeeper

到[Zookeeper官网](https://zookeeper.apache.org/releases.html#download)找到适合版本的下载链接。

在`bigdata01`节点上执行`wget`命令进行下载：
```shell
wget http://mirror.netinch.com/pub/apache/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz
```

#### 2. 安装Zookeeper

在`bigdata01`节点上创建安装目录：

```shell
mkdir -p /opt/software/
```

将下载的Zookeeper包拷贝到安装目录并解压：
```shell
cp apache-zookeeper-3.5.8-bin.tar.gz /opt/software/
```

解压：
```shell
cd /opt/software/
tar -zxvf apache-zookeeper-3.5.8-bin.tar.gz
rm apache-zookeeper-3.5.8-bin.tar.gz
mv apache-zookeeper* zookeeper
```

#### 3. 配置Zookeeper

在`Zookeper`安装目录下创建`data`和`logs`目录

```shell
cd /opt/software/zookeeper
mkdir data
mkdir logs
```

在`conf`目录下创建`zoo.cfg`配置文件：
```shell
cd /opt/software/zookeeper/conf
vim zoo.cfg
```

在`zoo.cfg`中配置如下内容，各个配置项的说明已在注释中列出：
```java
# tickTime表示服务器之间或客户端与服务器之间心跳的时间间隔，单位为毫秒
tickTime=2000
# follower与leader的初始连接心跳数
initLimit=10
# follower与leader请求和应答的最大心跳数
syncLimit=5
# 快照数据保存目录
dataDir=/opt/software/zookeeper/data
# 日志保存目录
dataLogDir=/opt/software/zookeeper/logs
# 客户端连接端口
clientPort=2181
# 客户端最大连接数,默认为60个
maxClientCnxns=60
# 默认为false，设置成true，zk将监听所有可用ip地址的连接
quorumListenOnAllIPs=false
# 服务器节点配置，格式为：
# server.<myid>=<host>:<leader和follower通信端口>:<选举端口>(observer节点最后加上:observer )
server.1=bigdata01:2888:3888
server.2=bigdata02:2888:3888
server.3=bigdata03:2888:3888
```

保存退出。

在`data`目录下创建`myid`文件：
```shell
vim /opt/software/zookeeper/data/myid
```

> **注意：**
> `myid`文件的内容必须与`zoo.cfg`里的配置保持一致。
> 例如：对于第一个节点`bigdata01`，在`zoo.cfg`里配置为：`server.1=bigdata01:2888:3888`，`server.1`表示`myid`为`1`，因此在`myid`文件中必须配置为`1`。

保存退出。

至此，在第一个节点`bigdata01`上的配置就完成了。接下来将整个`Zookeeper`安装目录同步到另外两个节点上去。

在`bigdata01`上执行如下命令：
```shell
rsync -az --delete /opt/software/zookeeper bigdata02:/opt/software/
rsync -az --delete /opt/software/zookeeper bigdata03:/opt/software/
```

> 如果`rsync`命令不存在，执行`apt-get install rsync`先进行安装。

接下来，由于每个节点的`myid`是不能相同的，因此，还需要修改同步过去的`myid`文件。分别登录另外两个节点，修改`myid`文件内容与`zoo.cfg`里的配置保持一致。也就是`bigdata02`节点的`myid`为`2`，`bigdata03`节点的`myid`为`3`。

最后，把整个`Zookeeper`的安装目录的`owner`修改为新创建的用户，分别在3个节点上执行命令：
```shell
chown -R bigdata:bigdata /opt/software/zookeeper
```

#### 4. 配置Zookeeper环境变量

分别在每个节点上配置`ZOOKEEPER_HOME`和`PATH`环境变量。

```shell
vim /etc/profile
```

添加如下内容：
```java
export ZOOKEEPER_HOME=/opt/software/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

```shell
source /etc/profile
```

#### 5. 启动Zookeeper

现在，可以启动`Zookeeper`了。

分别以用户`bigdata`登录每个节点，执行命令：

```shell
zkServer.sh start
```

### 四、验证Zookeeper集群

启动`Zookeeper`后，可以简单验证一下`Zookeeper`集群是否正常工作。

在任意节点执行命令：

```shell
zkCli.sh -server bigdata01:2181
```

如果一切正常，会进入`Zookeeper`的命令行，可以输入`ls /`查看，结果类似如下：
```shell
[zk: bigdata01:2181(CONNECTED) 0] ls /
[zookeeper]
```

如果出现以上结果，说明`Zookeeper`是正常工作的。







