---
title: 从零搭建solo个人博客网站
cover: true
top: true
date: 2018-07-11 21:39:01
group: frontend
permalink: build-solo-blog
categories: 前端
tags:
- Web
- 个人博客网站
keywords:
- 搭建个人博客
- solo个人博客
summary: 本人分享了我使用solo搭建个人博客网站的经验
---


### 前言

很久就想搭建一个自己的专属博客网站，用来记录与分享一些技术相关的文章，算做一个备忘录，以便把自己所学进行系统梳理，整理成文，方便以后回顾与巩固。本文记录了我从零搭建该博客网站，从购买服务器，到配置服务器，再到完成个人博客网站的搭建，总共两小时完成。 

要成功搭建一个网站，需要完成以下几个主要步骤： 

1. 购买服务器 

2. 购买域名及备案 

3. 安装依赖软件 

4. 安装博客程序 

5. 登录博客后台设置网站信息 

本文接下来将依次详细介绍每一个步骤。

### 1\. 购买服务器

我购买的阿里云服务器ECS，操作系统镜像选择的Ubuntu系统。以前也没有使用过其他的服务器，没法比较优劣。想来阿里云不会差，毕竟是大厂的。 

点击[这里](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=g89o8zwv)去阿里云官网上选择一款合适的服务器吧。购买后，登录阿里云管理控制台，进入`云服务器ECS`就可以看到你的服务器实例。实例会自动启动，几分钟就运行起来了。当你看到状态是运行中，表明已经启动成功了。你也可以看到这个实例的公网IP，你可以用远程SSH工具登录到这个IP进行服务器管理。 

另外，很重要很重要的是，你需要添加安全组规则，就是添加外部可以访问的端口。默认只开启了`22`端口。对于搭建网站，你必须要开通`80`端口，否则网站将无法访问。你可以开通其他端口，比如`MySQL`的端口，以便以后远程登录数据库查看数据。

### 2\. 购买域名与备案

服务器购买好后，你需要选择一个域名。这个不用多说，去阿里云旗下[万网](https://wanwang.aliyun.com/?spm=5176.8142029.388261.293.5e1f6d3exjjxUP)购买一个。 

购买后，你需要进行备案。备案对于中国大陆的服务器是必须的，否则就算你域名解析成功了，也是会被和谐掉的，所以去[这里](https://beian.aliyun.com/?spm=5176.100251.0.0.72014f158KiBjf)备案吧！

### 3\. 安装依赖软件

由于`Solo`是基于Java的开源博客系统，安装`Solo`之前，我们需要先安装如下依赖软件： 

1. Java 

2. MySQL 

3. Nginx

#### 3.1 安装Java

因为`Solo`是用Java开发的，我们要运行`Solo`必须的安装Java运行环境。在Oracle官网[下载页面](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载Linux版的JDK压缩包,然后上传到服务器。 

我在服务器上创建了一个新目录`/opt/java`，然后将压缩包拷贝到这个目录，然后解压：

```shell
tar -zxvf jdk-8u171-linux-x64.tar.gz
```

接下来设置环境变量，用vi编辑器打开`/etc/profile`文件：

```shell
vi /etc/profile
```

在文件开头添加如下内容：

```shell
JAVA_HOME=/opt/java/jdk1.8.0_171 
JRE_HOME=/opt/java/jdk1.8.0_171/jre 
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin 
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib 
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

然后保存别执行以下命令使其生效：

```shell
source /etc/profile
```

最后在任意目录执行下面命令来测试Java是否安装成功：

```shell
java -version
```

如果你看到类似下面的输出，说明已经安装成功了：

```shell
java version "1.8.0_171" 
Java(TM) SE Runtime Environment (build 1.8.0_171-b11) 
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

#### 3.2 安装MySQL

`Solo`默认使用的`H2`内存DB，我建议最好改用`MySQL`。

##### 3.2.1 安装MySQL

首先分别执行下面三条命令：

```shell
sudo apt-get install mysql-server 
sudo apt isntall mysql-client 
sudo apt install libmysqlclient-dev
```

安装过程中，要求设置`root`用户的密码，请一定记住这个密码。 

安装成功后可以通过下面的命令测试是否安装成功：

```shell
sudo netstat -tap | grep mysql
```

输出类似如下：

```shell
tcp6 0 0 localhost:mysql *:* LISTEN 19839/mysqld
```

你也可以执行以下命令测试是否可以进入`MySQL`：

```shell
mysql -uroot -p你的密码
```

##### 3.2.2 开启MySQL远程访问

`MySQL`安装后默认是没有打开远程访问的，从上面的输出可以看出，它只允许`localhost`也就是本机访问。 

我们可以编辑`/etc/mysql/mysql.conf.d/mysqld.cnf`文件：

```shell
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

将`bind-address = 127.0.0.1`注释掉：

```shell
#bind-address = 127.0.0.1
```

保存退出，然后执行如下命令进入`MySQL`：

```shell
mysql -uroot -p你的密码
```

然后执行以下命令进行授权：

```shell
grant all on *.* to root@'%' identified by '你的密码' with grant option; 
flush privileges;
```

然后执行`quit`命令退出`MySQL`，执行以下命令重启`MySQL`服务：

```shell
service mysql restart
```

此时，再次运行`ps`命令：

```shell
sudo netstat -tap | grep mysql
```

输出如下，你会看到它已经不再只是监听`localhost`了：

```shell
tcp6 0 0 [::]:mysql [::]:* LISTEN 19839/mysqld
```

现在你可以使用`MySQL`客户端测试一下是否可以从你的电脑访问服务器上的`MySQL`服务了。

#### 3.3 安装Ngnix

`Solo`会在自带的`Jetty`中运行，并默认监听`8080`端口，然而我们希望通过默认的`80`访问我们的网站，所以我们需要安装一个web server来做请求转发。 

Ubuntu系统中安装`Nginx`超简单，一条命令搞定：

```shell
sudo apt-get install nginx
```

安装好后`nginx`会自动启动，运行`ps`命令可以查看`nginx`进程:

```shell
> ps -ef | grep nginx 
root     20435     1 0 Jun25 ? 00:00:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on; 
www-data 20436 20435 0 Jun25 ? 00:00:01 nginx: worker process
```

接下来我们需要配置请求转发，打开`nginx`配置文件：

```shell
vi /etc/nginx/nginx.conf
```

在`http`节点内最后加上如下内容：

```shell
server {
        listen                  80;
        server_name             www.yglong.com;
        location / {
                proxy_pass              http://127.0.0.1:8080;
                proxy_redirect          off;
                proxy_set_header        Host $host;
                proxy_set_header        X-Real-IP $remote_addr;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

保存退出，重启`nginx`服务:

```shell
service nginx restart
```

### 4\. 安装博客程序

接下来就是安装博客程序了。有很多的开源博客程序，目前最火的应该是`WordPress`，用PHP开发的开源博客系统。但是由于我不熟悉PHP，所以选择了一款Java开源博客系统：[Solo](https://solo.b3log.org/)。 

点击这个[下载链接](https://pan.baidu.com/s/1dzk7SU)，或者通过上面的官网再进入Github找到下载链接，下载到Solo的war包。然后通过FTP工具上传到服务器。 

上传Solo包后，创建一个新目录`/opt/solo`，将Solo war包拷贝到这个目录下，然后解压：

```shell
jar -xvf solo-2.9.1.war
```

解压后，进入`latke.properties`文件：

```shell
vi /opt/solo/WEB-INF/classes/latke.properties
```

修改`serverHost`和`serverPort`：

```shell
#### Server ####
# Browser visit protocol
serverScheme=http
# Browser visit domain name
serverHost=www.yglong.com
# Browser visit port, 80 as usual, THIS IS NOT SERVER LISTEN PORT!
serverPort=80
```

保存并退出，进入`local.properties`文件：

```shell
vi /opt/solo/WEB-INF/classes/local.properties
```

注释掉H2 DB的配置，并配置MySQL：

```shell
#### H2 runtime ####
#runtimeDatabase=H2
#jdbc.username=root
#jdbc.password=
#jdbc.driver=org.h2.Driver
#jdbc.URL=jdbc:h2:~/solo_h2/db
#jdbc.pool=h2
 
#### MySQL runtime ####
runtimeDatabase=MYSQL
jdbc.username=root
jdbc.password=你的MySQL密码
jdbc.driver=com.mysql.jdbc.Driver
jdbc.URL=jdbc:mysql://localhost:3306/solo?useUnicode=yes&amp;characterEncoding=utf8
jdbc.pool=druid
```

保存退出。 接下来创建数据库，首先执行下面命令进入`MySQL`：

```shell
mysql -uroot -p你的密码
```

然后执行下面命令创建数据库，数据库名字与`local.properties`里配置的名字要一样：

```shell
CREATE DATABASE IF NOT EXISTS solo DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

退出`MySQL`，最后执行下面命令启动`Solo`：

```shell
nohup java -cp WEB-INF/lib/*:WEB-INF/classes org.b3log.solo.Starter &
```

### 5\. 登录博客后台设置网站信息

现在可以输入你的域名访问你的网站了。首次访问时，需要初始化网站。你需要设置你的管理员帐号，然后开始初始化，`Solo`会自动在`MySQL`中建立数据库表。初始化成功后就自动进入你的网站了。 

进入`Solo`后台管理控制台，进入`工具`->`偏好设定`，你可以修改你的网站名称等其他基本网站信息。 

就这样，你的网站已经基本搭建完成了。 

最后剩下的，也是经营个人网站最重要的，就是你需要坚持不断的发布有质量的，原创的好文！