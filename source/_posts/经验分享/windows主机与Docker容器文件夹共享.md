---
title: windows主机与Docker容器文件夹共享
cover: false
top: false
date: 2018-07-28 23:33:58
group: sharing
permalink: share-with-host-and-docker
categories: 经验分享
tags:
- Docker
- 容器
keywords:
- Docker容器共享
summary: 介绍如何将windows主机上的文件夹共享到Docker容器中
---


### 前言

在windows下使用`Docker ToolBox`时，有时候我们需要将主机某个文件夹共享到docker容器中，方便在windows主机与docker容器之间同步文件夹数据。但是我们都知道Docker ToolBox会启动`virtualbox`虚拟机，docker实际上是运行在虚拟机上的，而不是直接运行在windows主机上，所以没办法直接通过`docker run`的`-v` 参数实现主机与docker容器文件夹共享。 

本文将介绍如何在windows下实现主机与docker容器之间的文件夹共享。 主要有两大步骤： 

1. 配置windows主机与virtualbox虚拟机之间文件夹共享。 

2. 配置virtualbox虚拟机与docker容器之间文件夹共享。 

我们都知道`docker run`的`-v` 参数可以实现docker容器与宿主机之间目录共享。对于使用`Docker Toolbox`的情况，宿主机是并不是windows主机，而是在windows主机上通过virtualbox运行的虚拟机。因此，docker容器只能与虚拟机之间进行文件夹共享，要实现windows主机与docker容器之间文件夹共享，我们还需要配置windows主机与虚拟机之间文件夹共享，从而间接实现windows主机与docker容器之间文件夹共享。

### 一、 配置windows主机与virtualbox虚拟机之间文件夹共享

#### 1\. 在`VirtualBox`中添加共享文件夹

首先，打开Oracle VM VirtualBox管理器：

[![](http://wx1.sinaimg.cn/mw690/bd7db87egy1ftq0x1omh4j202102hmx8.jpg)](http://wx1.sinaimg.cn/mw690/bd7db87egy1ftq0x1omh4j202102hmx8.jpg) 

选择`Docker ToolBox`默认创建的`default` 虚拟机，点击工具栏的“设置”按钮，打开设置窗口： 

[![](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftq0x217gkj201k01jmwy.jpg)](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftq0x217gkj201k01jmwy.jpg) 

点击左侧菜单栏的“共享文件夹”打开共享文件夹设置面板。 

[![](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x33qtfj20uw0k5ju9.jpg)](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x33qtfj20uw0k5ju9.jpg) 

默认情况下，virtualbox已经配置了`c/Users`目录为共享文件夹，对应虚拟机里的共享目录为`/c/Users`。 

如果我们想共享其他文件夹，那么我们需要点击右侧的“添加共享文件夹”按钮： 

[![](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x2iitoj200s00u0jw.jpg)](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x2iitoj200s00u0jw.jpg) 

添加一个共享文件夹配置，如下： 

[![](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x45444j20k70c7ta1.jpg)](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftq0x45444j20k70c7ta1.jpg) 

如上图所示，我添加了一个名称为`MyBlog` 的共享文件夹，指向windows主机的`E:\\workspace\\MyBlog` 目录。同时，勾选“自动挂载”和“固定分配”。 

> PS：我共享该文件夹的目的是，以后在我本机上对我的blog的修改就可以动态同步到docker容器中，达到实时生效的目的 

[![](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftq0x4p81bj20k20c5gmj.jpg)](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftq0x4p81bj20k20c5gmj.jpg) 

点击`OK`保存。

#### 2\. 挂载共享文件夹

添加共享文件夹后，需要将其挂载到虚拟机上。 

##### 1) 自动挂载到默认目录 

由于我们前面添加共享文件夹时勾选了`自动挂载`，当我们重启虚拟机后，共享文件夹就会被自动挂载到虚拟机上，不用手动挂载。 

所以接下来我们启动`Docker Quickstart Terminal`，运行如下命令重启虚拟机：

```bash
docker-machine restart
```

重启后，运行如下命令进入虚拟机：

```bash
docker-machine ssh
```

进入虚拟机后，检查一下我们的共享文件夹是否自动挂载上了。共享文件夹会被自动挂载到根目录下，路劲为：`/<共享文件夹名称>` ，如下是我的例子：

```bash
docker@default:~$ ll /MyBlog
total 8
-rwxrwxrwx    1 docker   staff         1183 Jun 11 14:07 MyBlog.iml
-rwxrwxrwx    1 docker   staff         2263 Jun 11 14:05 pom.xml
drwxrwxrwx    1 docker   staff            0 Jun 11 14:05 src/
drwxrwxrwx    1 docker   staff            0 Jun 11 14:09 target/
```

可以看到，windows主机`E:\workspace\MyBlog` 里的内容已经共享到虚拟机`/MyBlog` 下了。 

##### 2) 挂载到指定目录

当然，我们也可以将共享文件夹挂载到虚拟机的指定目录下。 

通过`docker-machine ssh`命令进入虚拟机，然后在虚拟机中创建一个目录。我们将要把windows主机的共享文件夹挂载到该目录。例如，创建`/home/docker/MyBlog` :

```shell
mkdir /home/omc/MyBlog
```

我们现在查看该目录下是没有任何内容的：

```shell
docker@default:~$ ll /home/docker/MyBlog
total 0
```

接下来，将windows主机的共享文件夹挂载到该目录，运行如下命令：

```shell
sudo mount -t vboxsf MyBlog /home/docker/MyBlog
```

然后查看`/home/docker/MyBlog`目录：

```shell
docker@default:~$ ll /home/docker/MyBlog/
total 8
-rwxrwxrwx    1 root     root          1183 Jun 11 14:07 MyBlog.iml
-rwxrwxrwx    1 root     root          2263 Jun 11 14:05 pom.xml
drwxrwxrwx    1 root     root             0 Jun 11 14:05 src/
drwxrwxrwx    1 root     root             0 Jun 11 14:09 target/
```

可以看到已经成功挂载了！ 

但是，这种方式有一个弊端：**每当重启虚拟机后，新创建的目录会丢失。** 

我们可以通过如下方式**解决这个问题**： 

编辑`/mnt/sda1/var/lib/boot2docker/profile` 文件中：

```shell
sudo vi /mnt/sda1/var/lib/boot2docker/profile
```

在文件最后增加如下配置：
```shell
mkdir /home/docker/MyBlog
sudo mount -t vboxsf MyBlog /home/docker/MyBlog
```
保存退出虚拟机，执行如下命令重启虚拟机：

```bash
docker-machine restart
```

重启后，再次执行`docker-machine ssh` 进入虚拟机，查看是否自动挂载到我们指定的目录`/home/docker/MyBlog` ：

```shell
docker@default:~$ ll /home/docker/MyBlog/
total 8
-rwxrwxrwx    1 root     root          1183 Jun 11 14:07 MyBlog.iml
-rwxrwxrwx    1 root     root          2263 Jun 11 14:05 pom.xml
drwxrwxrwx    1 root     root             0 Jun 11 14:05 src/
drwxrwxrwx    1 root     root             0 Jun 11 14:09 target/
```

可以看到，挂载成功！

### 二、配置virtualbox虚拟机与docker容器之间文件夹共享

这个就很简单了，在执行`docker run`命令时，通过`-v`参数虚拟机某个目录挂载到docker容器中。 

如下，将前面在虚拟机中创建的`/home/docker/MyBlog`目录挂载到docker容器的`/usr/tmp/MyBlog`中：

```shell
docker run -v /home/docker/MyBlog:/usr/tmp/MyBlog -i -t centos /bin/bash
```

进入docker容器后，查看`/usr/tmp/MyBlog`目录：

```shell
root@d478e4c6c7f2:/# ll /usr/tmp/MyBlog/
total 20
drwxrwxrwx 1 root root 4096 Jun 11 14:09 ./
drwxr-xr-x 3 root root 4096 Jul 28 15:52 ../
drwxrwxrwx 1 root root 4096 Jun 27 13:52 .idea/
-rwxrwxrwx 1 root root 1183 Jun 11 14:07 MyBlog.iml*
-rwxrwxrwx 1 root root 2263 Jun 11 14:05 pom.xml*
drwxrwxrwx 1 root root    0 Jun 11 14:05 src/
drwxrwxrwx 1 root root    0 Jun 11 14:09 target/
```

可以看到，windows主机共享文件夹里的内容被成功同步到docker容器中了！