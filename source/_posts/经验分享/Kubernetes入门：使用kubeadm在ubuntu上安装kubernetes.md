---
title: Kubernetes入门：使用kubeadm在ubuntu上安装kubernetes
cover: false
top: false
date: 2018-12-13 17:49:03
group: sharing
permalink: kubernetes-get-started
categories: 经验分享
tags:
- Kubernetes
keywords:
- 安装Kubernetes
- Ubuntu上安装Kuberntes
- 使用kubeadm安装Kubernetes
summary: 本文介绍了使用 kubeadm 在 Ubuntu 上安装 Kubernetes
---


### 一、安装前准备

环境：`ubuntu 16.04`

```shell
$ sudo su

$ apt-get update

$ swapoff -a
```

#### 1. 安装`Docker`

```shell
$ sudo su

$ apt-get update

$ apt-get install -y docker.io
```

#### 2. 安装`OpenSSH`

如果没有安装`openssh`的话，执行下面命令安装：

```shell
$ sudo apt-get install openssh-server
```

#### 3. 安装`Kubernetes`工具

安装本文将要使用到个三个kubernetes工具：`kubelet`, `kubeadm`, `kubectl` 

首先准备安装这三个工具的条件：

```shell
$ apt-get update && apt-get install -y apt-transport-https curl

$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

$ apt-get update
```

执行下面命令安装三个工具：

```shell
$ apt-get install -y kubelet kubeadm kubectl
```

### 二、配置`Kubernetes`

```shell
$ vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

在所有`Environment`后面添加一行：

```
Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"
```

> 注意：以上`步骤一`和`步骤二`需要在`所有node`上执行，包括`master`和各个`node`节点上。

### 三、配置`Master`

以下步骤仅在`master`节点上执行。 

首先使用`kuberadm init`初始化kubernetes。

```shell
$ kubeadm init --apiserver-advertise-address=192.168.0.12 --pod-network-cidr=192.168.0.0/16
```

`--apiserver-advertise-address`为`master`节点的`IP地址`。 

上面命令执行成功后，执行以下命令配置`kubectl`：

```shell
$ mkdir -p $HOME/.kube

$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

接下来可以运行`kubectl`命令检查kubernetes是否成功`初始化`：

```shell
$ kubectl get pods -o wide --all-namespaces
```

接下来配置`master网络`，可以选择`Calico`或者`Flannel`。 

如果用`Calico`，执行以下命令：

```shell
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/etcd.yaml

$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/rbac.yaml

$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/calico.yaml
```

如果用`Flannel`，执行下面命令：

```shell
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

默认情况下，`master`上是`不允许创建pod`的，如果想在master上也可以创建pod，可以执行以下命令开启：

```shell
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

### 四、配置`Node`节点

`Node节点`几乎不需要任何配置，只要安装好kubernetes三个工具后，执行`kubeadm join`把node节点加入kubernetes集群即可：

```shell
$ kubeadm join 192.168.0.12:6443 --token xmbbp0.qu9z2d7qw5b9m855 --discovery-token-ca-cert-hash sha256:2eb0e8dde0df05555222ade14fb15f88d38b4a0d433fe50d3c5614f26f9109ff
```

这个命令在master上执行`kubeadm init`时会给出，包含了具体的`token`和`ca-cert`，请按实际输出的信息执行`join`命令。