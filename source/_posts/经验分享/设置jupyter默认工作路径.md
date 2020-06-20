---
title: 设置jupyter默认工作路径
cover: false
top: false
date: 2019-02-12 18:25:48
group: sharing
permalink: set-jupyter-workspace
categories: 经验分享
tags:
- Jupyter
keywords:
- 设置jupyter默认工作路径
summary: 本文介绍了如何设置jupyter默认工作路径
---

安装好`Anaconda`后，启动`Jupyter notebook`时，通常会默认工作目录，所谓`工作目录`，就是当你创建新的`notebook`时文件的`默认存放路径`。一般情况下，默认的工作目录是：

```
C:\Users\<用户名>
```

那么如何将这个默认的工作目录修改为自定义的目录呢？

### Windows

在`Windows`系统中，最简单的办法是直接修改`jupyter notebook`的`快捷方式`。 

在`Windows`的开始菜单里找到`Jupyter Notebook`的快捷方式（一般在`Anaconda3 (64-bit)`目录下），然后在右键菜单中选择属性，在`快捷方式`选项卡中的`目标(T)`里的最后的`%USERPROFILE%`修改为你的工作目录即可。 

**注意：**

-   你的`工作目录`需要手动创建好，否则启动会失败。
-   你的`工作目录`需要用双引号括起来，如：`"E:\workspace\notebook"`

修改好后，应用及确定，然后点击快捷方式启动`Jupyter Notebook`，你会看到默认的工作目录已经变为你自己的工作目录了。

### Linux

在`Linux`系统中，我们可以通过修改配置文件`~/.jupyter/jupyter_notebook_config.py` 如果你没有找到该配置文件，可以执行如下命令生成该配置文件：

```shell
jupyter notebook --generate-config
```

修改该配置文件的以下属性：

```
c.NotebookApp.notebook_dir = '/root/workspace/projects'
```

修改后，当你运行`jupyter notebook`重新启动`notebook`是会看到默认工作目录已经变成新的目录了。
