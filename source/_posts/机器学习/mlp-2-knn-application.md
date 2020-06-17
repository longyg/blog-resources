---
title: 《机器学习实战》读书笔记之二：k-近邻算法应用案例
cover: true
top: true
mathjax: true
date: 2018-07-21 22:56:42
group: ml
permalink: mlp-2-knn-application
categories: 机器学习
tags:
- 机器学习
- 算法
keywords:
- KNN算法应用案例
summary: 介绍KNN算法的两个应用案例
---


### 应用案例一：使用k-近邻算法提高约会网站配对成功率

#### 1\. 案例说明

这个例子是海伦想通过在线约会网站寻找适合自己的约会对象，她认为有三种类型的约会对象：

-   不喜欢的人
-   魅力一般的人 （也就是可能是她喜欢的人）
-   极具魅力的人 （也就是她喜欢的人）

她想通过自己收集的数据对所有人进行归类，找到适合自己的约会对象。她收集的数据包含3个特征：

-   每年获得的飞行常客里程数 （我觉得这个可能说明这个人经常出去旅行，海伦喜欢旅行）
-   玩视频游戏所耗时间百分比 （海伦喜欢自由时间多的人，这样就有时间陪她约会，哪怕是视频？脑补一下！）
-   每周消费的冰淇淋公升数 （这个... 说明海伦想找个吃货...只能说明她自己也是个吃货吧！）

海伦收集的数据存储在一个文本文件中，每一行是是一个样本，每个样本前三个数据是3个特征值，最后一个数据是海伦对其进行的分类。

-   `largeDoses`: 极具魅力
-   `smallDoses`：魅力一般
-   `didntLike`：不喜欢

示例数据如下：

```
40920    8.326976    0.953952    largeDoses
14488    7.153469    1.673904    smallDoses
26052    1.441871    0.805124    didntLike
75136    13.147394    0.428964    didntLike
38344    1.669788    0.134296    didntLike
72993    10.141740    1.032955    didntLike
35948    6.830792    1.213192    largeDoses
42666    13.276369    0.543880    largeDoses
67497    8.631577    0.749278    didntLike
...
```

海伦总共收集了1000个样本，你可以从这里[下载数据文件](https://github.com/longyg/Machine-Learning-Practice/blob/master/kNN/Halen%20date/datingTestSet.txt)

#### 2\. 解析数据

有了数据文件后，我们需要解析该文件，将其解析为分类器函数可以接收的格式。还记得我们的分类器函数`classifier()`接收什么样的输入数据格式吧？包含两个参数：

1.  特征矩阵：`dataSet`
2.  分类标签列表(也可认为是向量)：`labels`

为了从数据文件中提取出这两个输入参数，我们实现以下函数`file2matrix()`,以文件名为参数：

```python
from numpy import *
def file2matrix(filename):
    fr = open(filename)
    arrayOLines = fr.readlines()
    numberOfLines = len(arrayOLines)
    returnMat = zeros((numberOfLines, 3))
    classLabelVector = []
    index = 0
    for line in arrayOLines:
        line = line.strip()
        listFromLine = line.split('\t')
        returnMat[index, :] = listFromLine[0: 3]
        if listFromLine[-1] == 'didntLike':
            classLabelVector.append(1)
        elif listFromLine[-1] == 'smallDoses':
            classLabelVector.append(2)
        elif listFromLine[-1] == 'largeDoses':
            classLabelVector.append(3)
        index += 1
    return returnMat, classLabelVector
```

将数据文件放到该程序代码同一个目录下，名为：`datingTestSet.txt`，然后调用该函数看看解析结果：

```python
filename = 'datingTestSet.txt'
dataSet, labels = file2matrix(filename)
print(dataSet)
```

```
[[4.0920000e+04 8.3269760e+00 9.5395200e-01]
 [1.4488000e+04 7.1534690e+00 1.6739040e+00]
 [2.6052000e+04 1.4418710e+00 8.0512400e-01]
 ...
 [2.6575000e+04 1.0650102e+01 8.6662700e-01]
 [4.8111000e+04 9.1345280e+00 7.2804500e-01]
 [4.3757000e+04 7.8826010e+00 1.3324460e+00]]
```

```python
print(labels)
```
```
[3, 2, 1, 1, 1, 1, 3, 3, 1, 3, 1, 1, 2, 1, 1, 1, 1, 1, 2, 3, 2, 1, 2, 3, 2, 3, 2, 3, 2, 1, 3, 1, 3, 1, 2, 1, 1, 2, 3, 3, 1, 2, 3, 3, 3, 1, 1, 1, 1, 2, 2, 1, 3, 2, 2, 2, 2, 3, 1, 2, 1, 2, 2, 2, 2, 2, 3, 2, 3, 1, 2, 3, 2, 2, 1, 3, 1, 1, 3, 3, 1, 2, 3, 1, 3, 1, 2, 2, 1, 1, 3, 3, 1, 2, 1, 3, 3, 2, 1, 1, 3, 1, 2, 3, 3, 2, 3, 3, 1, 2, 3, 2, 1, 3, 1, 2, 1, 1, 2, 3, 2, 3, 2, 3, 2, 1, 3, 3, 3, 1, 3, 2, 2, 3, 1, 3, 3, 3, 1, 3, 1, 1, 3, 3, 2, 3, 3, 1, 2, 3, 2, 2, 3, 3, 3, 1, 2, 2, 1, 1, 3, 2, 3, 3, 1, 2, 1, 3, 1, 2, 3, 2, 3, 1, 1, 1, 3, 2, 3, 1, 3, 2, 1, 3, 2, 2, 3, 2, 3, 2, 1, 1, 3, 1, 3, 2, 2, 2, 3, 2, 2, 1, 2, 2, 3, 1, 3, 3, 2, 1, 1, 1, 2, 1, 3, 3, 3, 3, 2, 1, 1, 1, 2, 3, 2, 1, 3, 1, 3, 2, 2, 3, 1, 3, 1, 1, 2, 1, 2, 2, 1, 3, 1, 3, 2, 3, 1, 2, 3, 1, 1, 1, 1, 2, 3, 2, 2, 3, 1, 2, 1, 1, 1, 3, 3, 2, 1, 1, 1, 2, 2, 3, 1, 1, 1, 2, 1, 1, 2, 1, 1, 1, 2, 2, 3, 2, 3, 3, 3, 3, 1, 2, 3, 1, 1, 1, 3, 1, 3, 2, 2, 1, 3, 1, 3, 2, 2, 1, 2, 2, 3, 1, 3, 2, 1, 1, 3, 3, 2, 3, 3, 2, 3, 1, 3, 1, 3, 3, 1, 3, 2, 1, 3, 1, 3, 2, 1, 2, 2, 1, 3, 1, 1, 3, 3, 2, 2, 3, 1, 2, 3, 3, 2, 2, 1, 1, 1, 1, 3, 2, 1, 1, 3, 2, 1, 1, 3, 3, 3, 2, 3, 2, 1, 1, 1, 1, 1, 3, 2, 2, 1, 2, 1, 3, 2, 1, 3, 2, 1, 3, 1, 1, 3, 3, 3, 3, 2, 1, 1, 2, 1, 3, 3, 2, 1, 2, 3, 2, 1, 2, 2, 2, 1, 1, 3, 1, 1, 2, 3, 1, 1, 2, 3, 1, 3, 1, 1, 2, 2, 1, 2, 2, 2, 3, 1, 1, 1, 3, 1, 3, 1, 3, 3, 1, 1, 1, 3, 2, 3, 3, 2, 2, 1, 1, 1, 2, 1, 2, 2, 3, 3, 3, 1, 1, 3, 3, 2, 3, 3, 2, 3, 3, 3, 2, 3, 3, 1, 2, 3, 2, 1, 1, 1, 1, 3, 3, 3, 3, 2, 1, 1, 1, 1, 3, 1, 1, 2, 1, 1, 2, 3, 2, 1, 2, 2, 2, 3, 2, 1, 3, 2, 3, 2, 3, 2, 1, 1, 2, 3, 1, 3, 3, 3, 1, 2, 1, 2, 2, 1, 2, 2, 2, 2, 2, 3, 2, 1, 3, 3, 2, 2, 2, 3, 1, 2, 1, 1, 3, 2, 3, 2, 3, 2, 3, 3, 2, 2, 1, 3, 1, 2, 1, 3, 1, 1, 1, 3, 1, 1, 3, 3, 2, 2, 1, 3, 1, 1, 3, 2, 3, 1, 1, 3, 1, 3, 3, 1, 2, 3, 1, 3, 1, 1, 2, 1, 3, 1, 1, 1, 1, 2, 1, 3, 1, 2, 1, 3, 1, 3, 1, 1, 2, 2, 2, 3, 2, 2, 1, 2, 3, 3, 2, 3, 3, 3, 2, 3, 3, 1, 3, 2, 3, 2, 1, 2, 1, 1, 1, 2, 3, 2, 2, 1, 2, 2, 1, 3, 1, 3, 3, 3, 2, 2, 3, 3, 1, 2, 2, 2, 3, 1, 2, 1, 3, 1, 2, 3, 1, 1, 1, 2, 2, 3, 1, 3, 1, 1, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 2, 2, 2, 3, 1, 3, 1, 2, 3, 2, 2, 3, 1, 2, 3, 2, 3, 1, 2, 2, 3, 1, 1, 1, 2, 2, 1, 1, 2, 1, 2, 1, 2, 3, 2, 1, 3, 3, 3, 1, 1, 3, 1, 2, 3, 3, 2, 2, 2, 1, 2, 3, 2, 2, 3, 2, 2, 2, 3, 3, 2, 1, 3, 2, 1, 3, 3, 1, 2, 3, 2, 1, 3, 3, 3, 1, 2, 2, 2, 3, 2, 3, 3, 1, 2, 1, 1, 2, 1, 3, 1, 2, 2, 1, 3, 2, 1, 3, 3, 2, 2, 2, 1, 2, 2, 1, 3, 1, 3, 1, 3, 3, 1, 1, 2, 3, 2, 2, 3, 1, 1, 1, 1, 3, 2, 2, 1, 3, 1, 2, 3, 1, 3, 1, 3, 1, 1, 3, 2, 3, 1, 1, 3, 3, 3, 3, 1, 3, 2, 2, 1, 1, 3, 3, 2, 2, 2, 1, 2, 1, 2, 1, 3, 2, 1, 2, 2, 3, 1, 2, 2, 2, 3, 2, 1, 2, 1, 2, 3, 3, 2, 3, 1, 1, 3, 3, 1, 2, 2, 2, 2, 2, 2, 1, 3, 3, 3, 3, 3, 1, 1, 3, 2, 1, 2, 1, 2, 2, 3, 2, 2, 2, 3, 1, 2, 1, 2, 2, 1, 1, 2, 3, 3, 1, 1, 1, 1, 3, 3, 3, 3, 3, 3, 1, 3, 3, 2, 3, 2, 3, 3, 2, 2, 1, 1, 1, 3, 3, 1, 1, 1, 3, 3, 2, 1, 2, 1, 1, 2, 2, 1, 1, 1, 3, 1, 1, 2, 3, 2, 2, 1, 3, 1, 2, 3, 1, 2, 2, 2, 2, 3, 2, 3, 3, 1, 2, 1, 2, 3, 1, 3, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 2, 2, 2, 2, 2, 1, 3, 3, 3]
```

现在我们已经成功解析了输入数据文件，接下来我们可以对这些数据进行一些初步分析。

#### 3\. 分析数据

在机器学习中，要对大规模的数据集进行分析，我们一般采用图形化的方式来展示数据，这样会比较直观，通过图形可以方便的找出一些数据模式（即规律）。 下面我们将前面的数据集图形化，实现以下函数`showDataSet()`，将`file2matrix()`函数返回的两个结果作为参数：

```python
import matplotlib
import matplotlib.pyplot as plt
 
def showDataSet(dataSet):
    fig, axs = plt.subplots(nrows=2, ncols=2, sharex=False, sharey=False, figsize=(15,10))  
    
    axs[0][0].scatter(dataSet[:, 0], dataSet[:, 1], 15.0*array(labels), 15.0*array(labels))
    axs[0][0].set_title('每年获得的飞行常客里程数与玩视频游戏所消耗时间百分比')
    axs[0][0].set_xlabel('每年获得的飞行常客里程数')
    axs[0][0].set_ylabel('玩视频游戏所消耗时间百分比')
    
    axs[0][1].scatter(dataSet[:, 0], dataSet[:, 2], 15.0*array(labels), 15.0*array(labels))
    axs[0][1].set_title('每年获得的飞行常客里程数与每周消费的冰激淋公升数')
    axs[0][1].set_xlabel('每年获得的飞行常客里程数')
    axs[0][1].set_ylabel('每周消费的冰激淋公升数')
    
    axs[1][0].scatter(dataSet[:, 1], dataSet[:, 2], 15.0*array(labels), 15.0*array(labels))
    axs[1][0].set_title('玩视频游戏所消耗时间百分比与每周消费的冰激淋公升数')
    axs[1][0].set_xlabel('玩视频游戏所消耗时间百分比')
    axs[1][0].set_ylabel('每周消费的冰激淋公升数')
    
    plt.rcParams['font.sans-serif'] = ['FangSong']
    plt.rcParams['axes.unicode_minus'] = False
    plt.show()
```
```python
showDataSet(dataSet)
```

[![](http://wx3.sinaimg.cn/mw690/bd7db87egy1fthw8ru8tkj20oz0gstfg.jpg)](http://wx3.sinaimg.cn/mw690/bd7db87egy1fthw8ru8tkj20oz0gstfg.jpg) 

我们可以从上面的图中看出一些规律。上面三个图分别取了三个特征中的两个进行绘制，这样更容易从二维图形中区分数据的类别。大概可以猜测一下，海伦喜欢的人，每年至少有一定的飞行记录，不能太少也不能太多，太少说明很少出去旅行，太多说明有可能是因公常年外出。不过这纯属我个人猜测，大家可以自行脑补！

#### 4\. 预处理数据

前面我们的电影分类例子中，特征是二维的，计算距离使用的是二维平面中两点之间的距离公式。但是这里海伦收集的数据是三维的，那么我们如何计算样本之间的距离呢？ 

实际上，我们可以扩展前面的二维平面两点距离公式到任意维度，采用欧几里得距离公式。

假设一个`n`维空间的两个点表示为：

-   点1: $(x_1, x_2, x_3, ..., x_n)$
-   点2: $(y_1, y_2, y_3, ..., y_n)$

那么点1与点2的距离为： 

$$\sqrt{(x_1 - y_1)^{2} + (x_2 - y_2)^{2} + (x_3 - y_3)^{2} + ... + (x_n - y_n)^{2}}$$ 

因此我们将上面公式应用于海伦收集的数据，例如我们可以计算数据文件中前两个样本之间的距离，样本数据为：

```
40920    8.326976    0.953952    largeDoses
14488    7.153469    1.673904    smallDoses
```

应用上面的公式为： 

$$\sqrt{(40920 - 14488)^{2} + (8.326976 - 7.153469)^{2} + (0.953952 - 1.673904)^{2}}$$

从上面公式可以发现，方程中数字差值最大的属性对计算结果影响最大。因为每年飞行里程数的数值一般情况下都是几百上千，甚至上万，而另外两个特征的数值一般情况下都很小。也就是说，每年飞行里程数对于计算结果的影响远远大于另外两个特征。 

如果我们直接使用这样的样本数据计算，会导致结果几乎由三个特征中的一个（也就是每年飞行里程数）决定了，而另外两个特征几乎不会影响计算结果。 

这明显是不可接受的，因为海伦认为这三个特征是同等重要的，因此这个特征应该具有相同权重。 

对于这种情况，我们需要对样本数据进行预处理，通常采用的方法是将数值进行归一化，比如将每个特征的取值范围处理为`0`到`1`或者`-1`到`1`之间。 

下面的公式可以将任意取值范围的数值转化为0到1区间内的值：

```python
newValue = (oldValue - min) / (max - min)
```

其中`min`和`max`分别是数据集中的最小特征值和最大特征值，`oldValue`是数据集中原始特征值，`newValue`是进行归一化处理后的结果值，我们将使用这个预处理之后的数值进行分类计算。 

现在我们定义一个`autoNorm()`函数来实现上面的归一化公式:

```python
def autoNorm(dataSet):
    minVals = dataSet.min(0)
    maxVals = dataSet.max(0)
    ranges = maxVals - minVals
    m = dataSet.shape[0]
    normDataSet = dataSet - tile(minVals, (m, 1))
    normDataSet = normDataSet / tile(ranges, (m, 1))
    return normDataSet, ranges, minVals
 
ormDataSet, ranges, minVals = autoNorm(dataSet)
 
print(ormDataSet)
```
```
[[0.44832535 0.39805139 0.56233353]
 [0.15873259 0.34195467 0.98724416]
 [0.28542943 0.06892523 0.47449629]
 ...
 [0.29115949 0.50910294 0.51079493]
 [0.52711097 0.43665451 0.4290048 ]
 [0.47940793 0.3768091  0.78571804]]
```

```python
print(ranges)
```

```
[9.1273000e+04 2.0919349e+01 1.6943610e+00]
```

```python
print(minVals)
```

```
[0.       0.       0.001156]
```

`autoNorm()`函数中,首先使用numpy矩阵的`min()`和`max()`函数得到每个特征的最小和最大值，分别放在变量`minVals`和`maxVals`中。 

`dataSet.min(0)`是从返回`dataSet`矩阵每一列的最小值，`dataSet.max(0)`返回矩阵每一列的最大值。 

因为每一列的最小值和最大值只有1个，而我们有三个特征值也就是三列，所以`minVals`和`maxVals`变量存放的是`1x3`的矩阵：

```python
minVals = dataSet.min(0)
print(minVals)
```

```
[0.       0.       0.001156]
```

```python
maxVals = dataSet.max(0)
print(maxVals)
```

```
[9.1273000e+04 2.0919349e+01 1.6955170e+00]
```

同理，`ranges`也是`1x3`的，因为两个矩阵之差得到的是相同维度的矩阵：

```python
ranges = maxVals - minVals
print(ranges)
```

```
[9.1273000e+04 2.0919349e+01 1.6943610e+00]
```

根据上面的归一化公式，我们需要用原始特征值减去最小值，然后除以每个特征的取值范围（即最大值减最小值）。由于我们的特征矩阵`dataSet`是一个`1000x3`的矩阵，因此我们需要将`minVals`和`ranges`重复变成同样大小的矩阵，这里我们依然使用了numpy的`tile()`函数来重复矩阵。我们需要首先得到`dataSet`的大小，通过矩阵的`shape`属性获取矩阵大小：

```python
m = dataSet.shape\[0\]
print(m)
```

```
1000
```

然后重复最小特征矩阵`minVals` `m`次，注意是在行方向上重复`m`次，从而构造`1000x3`的矩阵：

```python
tmpMat = tile(minVals, (m, 1))
print(tmpMat)
```

```
[[0.       0.       0.001156]
 [0.       0.       0.001156]
 [0.       0.       0.001156]
 ...
 [0.       0.       0.001156]
 [0.       0.       0.001156]
 [0.       0.       0.001156]]
```

接下来用原始特征值减去最小值：

```python
normDataSet = dataSet - tile(minVals, (m, 1))
print(normDataSet)
```

```
[[4.0920000e+04 8.3269760e+00 9.5279600e-01]
 [1.4488000e+04 7.1534690e+00 1.6727480e+00]
 [2.6052000e+04 1.4418710e+00 8.0396800e-01]
 ...
 [2.6575000e+04 1.0650102e+01 8.6547100e-01]
 [4.8111000e+04 9.1345280e+00 7.2688900e-01]
 [4.3757000e+04 7.8826010e+00 1.3312900e+00]]
```

然后重复`ranges`矩阵`m`次，并用上一步求出的原始特征值与最小值的差值除以取值范围：

```python
normDataSet = normDataSet / tile(ranges, (m, 1))
print(normDataSet)
```

```
[[0.44832535 0.39805139 0.56233353]
 [0.15873259 0.34195467 0.98724416]
 [0.28542943 0.06892523 0.47449629]
 ...
 [0.29115949 0.50910294 0.51079493]
 [0.52711097 0.43665451 0.4290048 ]
 [0.47940793 0.3768091  0.78571804]]
```

我们已经完成了数据的预处理。对数据进行处理是机器学习中训练算法很关键的一步，因为数据的正确性直接影响算法的正确性。

#### 5\. 测试分类器

现在，我们已经对原始数据进行了预处理，接下来，我们可以用这些数据来测试我们前面实现的`kNN`算法分类器了。 

这里再一次将我们的分类器算法列出来，也就是我们前面详细讲解过的`classifier()`函数：

```python
from numpy import *
import operator
import sys
 
def classifier(inX, dataSet, labels, k):
    dataSetSize = dataSet.shape[0]
    diffMat = tile(inX, (dataSetSize, 1)) - dataSet
    sqDiffMat = diffMat**2
    sqDistances = sqDiffMat.sum(axis=1)
    distances = sqDistances**0.5
    sortedDistIndicies = distances.argsort()
    classCount = {}
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1
    sortedClassCount = sorted(classCount.items(), key=operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]
```

我们需要从数据集中选出一部分作为分类器训练数据，一部分作为测试数据，用于测试分类器的正确率。一般情况下，我们使用错误率来检测分类器的效果。错误率就是分类器给出错误结果的次数除以测试样本总数，完美分类器的错误率为`0`，而错误率为`1`的分类器根本不能给出任何正确的结果。因此我们希望我们的分类器的错误率越小越好。 

现在我们定义一个`datingClassTest()`函数来计算分类器应用于海伦收集的约会数据集的错误率：

```python
def datingClassTest():
    testRatio = 0.1
    datingDataMat, datingLabels = file2matrix('datingTestSet.txt')
    normMat, ranges, minVals = autoNorm(datingDataMat)
    m = normMat.shape[0]
    numTestVecs = int(m * testRatio)
    trainDataMat = normMat[numTestVecs:m,:]
    trainLabels = datingLabels[numTestVecs:m]
    k = 3
    errorCount = 0.0
    for i in range(numTestVecs):
        testIdata = normMat[i, :]
        classResult = classifier(testIdata, trainDataMat, trainLabels, k)
        print("分类器预测结果是: ", classResult, " 实际结果是: ", datingLabels[i])
        if (classResult != datingLabels[i]):
            errorCount += 1.0
    print("总的错误率是: ", errorCount / float(numTestVecs))
```

```python
datingClassTest()
```

```
分类器预测结果是:  3  实际结果是:  3
分类器预测结果是:  2  实际结果是:  2
分类器预测结果是:  1  实际结果是:  1
分类器预测结果是:  1  实际结果是:  1
分类器预测结果是:  1  实际结果是:  1
...
分类器预测结果是:  2  实际结果是:  2
分类器预测结果是:  1  实际结果是:  1
分类器预测结果是:  3  实际结果是:  1
总的错误率是:  0.05
```

可以看到，分类器对于海伦收集的数据的分类错误率是`5%`，这个结果还不错。我们可以改变测试数据比例变量`testRatio`的值，可以发现错误率会随着变量值的变化而变化。 

由于分类器的错误率还不错，那么我们现在可以输入任意未知约会对象的属性，由分类软件来帮助海伦判断对象是否是海伦喜欢的类型。 

那么我们来设计一个分类软件，允许用户输入某个人的信息，程序返回预测结果：

```python
def classifyPerson():
    resultList = ['你应该不喜欢他', '你可能会有一点喜欢他', '他应该是你喜欢的人']
    percentTats = float(input("玩视频游戏所消耗时间百分比？"))
    ffMiles = float(input("每年获得的飞行常客里程数？"))
    iceCream = float(input("每周消费的冰淇淋公升数？"))
    datingDataMat, datingLabels = file2matrix('datingTestSet.txt')
    normMat, ranges, minVals = autoNorm(datingDataMat)
    inArr = array([ffMiles, percentTats, iceCream])
    preProcessInArr = (inArr - minVals) / ranges
    classifierResult = classifier(preProcessInArr, normMat, datingLabels, 3)
    print(">> 预测结果：", resultList[classifierResult - 1])
```

```python
classifyPerson()
```

```
玩视频游戏所消耗时间百分比？30
每年获得的飞行常客里程数？1500
每周消费的冰淇淋公升数？2
>> 预测结果： 他应该是你喜欢的人
```

我们已经基于机器学习的`kNN`算法构建了一个完整的应用程序，它完全可以被应用于实际中，只是对于现实情况，可能不仅仅三个特征，可能会有很多特征属性来区分一个约会对象的类型。但是不管有多少特征，我们都可以采用同样的分类器来进行预测。 

从[这里下载](https://github.com/longyg/Machine-Learning-Practice/blob/master/kNN/Halen%20date/dating.py)完整代码实现dating.py。

### 应用案例二：手写识别系统

#### 1\. 案例说明

本案例将实现一个手写识别系统，为了简单起见，本案例只实现了识别数字`0`到`9`。在应用我们的分类器之前，为了方便将图像数据处理成分类器识别的数据集，我们将图像处理成`32x32`的文本文件。什么意思？就是说我们的每个文本文件包含`32`行，`32`列，每个索引位置由`0`或`1`填充，代表一个手写数字的图像。 

你可以在[这里下载](https://github.com/longyg/Machine-Learning-Practice/tree/master/kNN/%E6%89%8B%E5%86%99%E8%AF%86%E5%88%AB)我们已经处理好的所有文件，然后打开任意一个文件进行观察。例如下面是一个手写数字3的图像处理成的文本文件内容（文件`3_97.txt`）：

```
00000000011111111100000000000000
00000000011111111111000000000000
00000000111111111111110000000000
00000000111111111111111000000000
00000000011100000011111100000000
00000000000000000000111110000000
00000000000000000000111110000000
00000000000000000000111110000000
00000000000000000001111110000000
00000000000000000011111100000000
00000000000000000111111100000000
00000000000011111111111000000000
00000001111111111111100000000000
00000011111111111110000000000000
00000011111111111110000000000000
00000001111111111111100000000000
00000001111111111111100000000000
00000000000000111111111000000000
00000000000000000111111100000000
00000000000000000001111100000000
00000000000000000000111111000000
00000000000000000000011111000000
00000000000000000000011111000000
00000000000000000000001111100000
00000000000000000000011111100000
00000000111000000000011110000000
00000000111000000001111110000000
00000000111100000111111100000000
00000000111111111111111000000000
00000000111111111111100000000000
00000000011111111111000000000000
00000000001111111000000000000000
```

如果你仔细看这个文本内容，把所有`1`连接起来观察，可以看出来它就是数字`3`。同样你可以打开其他任意文件进行观察。

#### 2\. 处理数据

在前面提供的[下载链接](https://github.com/longyg/Machine-Learning-Practice/tree/master/kNN/%E6%89%8B%E5%86%99%E8%AF%86%E5%88%AB)中，包含两类文件分别放在两个不同目录下： 

1. 目录`trainingDigits`下包含的是用于训练的样本文件，大约2000个例子，包含了所有从0到9的数字样本，其中每个数字大约200个样本。 

2. 目录`testDigits`下包含的是用于测试的样本文件，大约有900个。 

由于每个文本文件存储的是`32x32`的矩阵数据，我们需要将其格式化处理为一个向量，这样才能使用我们的分类器。由于`32x32 = 1024`，因此我们将其转换为`1x1024`的向量。 

所以我们首先定义一个函数`img2vector()`，将文本文件`32x32`的数据内容转换为`1x1024`的向量。

```python
import numpy as np
 
def img2vector(filename):
    vector = np.zeros((1, 1024))
    f = open(filename)
    for i in range(32):
        line = f.readline()
        for j in range(32):
            vector[0, 32*i+j] = int(line[j])
    return vector
```

该函数首先使用numpy的`zeros()`函数创建一个`1x1024`的数组（即向量），全部由`0`填充。然后打开输入文件，循环读取文件包含数据的前`32`行，并将每行包含数据的前`32`个字符（即前32列）分别填充到由`zeros()`函数创建的`1x1024`数组中，这样就将文本文件32行32列的所有数据存储到了`1x1024`的数组中，最后返回该`1x1024`数组。

#### 3\. 测试分类器

接下来我们开始测试我们之前实现的`kNN`算法分类器，因此我们定义一个测试函数`handwritingClassTest()`：

```python
import os
 
def handwritingClassTest():
    labels = []
    trainingFileList = os.listdir(trainingDir)
    m = len(trainingFileList)
    trainingMat = np.zeros((m, 1024))
    for i in range(m):
        fileName = trainingFileList[i]
        fileStr = fileName.split('.')[0]
        classNumber = int(fileStr.split('_')[0])
        labels.append(classNumber)
        trainingMat[i, :] = img2vector(os.path.join(trainingDir, fileName))
    testFileList = os.listdir(testDir)
    errorCount = 0.0
    mTest = len(testFileList)
    for i in range(mTest):
        fileName = testFileList[i]
        fileStr = fileName.split('.')[0]
        classNumber = int(fileStr.split('_')[0])
        vectorUnderTest = img2vector(os.path.join(testDir, fileName))
        result = classifier(vectorUnderTest, trainingMat, labels, 3)
        print("分类器预测结果: %d, 实际答案是: %d" % (result, classNumber))
        if result != classNumber: 
            errorCount += 1.0
    print("\n预测错误数: %d" % errorCount)
    print("\n错误率为: %f" % (errorCount/float(mTest)))
```

程序使用`os`模块的`listdir()`函数分别读取`trainingDigits`下的训练样本文件，以及`testDigits`下的测试样本文件，分别存储在不同的列表中。 

根据训练样本列表的大小`m`构造一个`mx1024`的矩阵，也就是训练数据集，用于存储所有训练样本数据，其中每一行为一个图像的数据。程序会调用前面实现的`img2vector()`函数循环将每个训练文本文件转换为`1x1024`的向量，然后填充到`mx1024`的矩阵中的对应行。 

注意，比较特殊的是，每个文本文件的文件名是固定格式，包含了图像的标签信息。例如前面例子打开的`3_97.txt`，从文件名可以看出，这个文件的label是`3`，它是数字`3`的第`97`个样本。因此程序中通过解析文件名获得所有数据的`label`。 

类似地，程序使用同样的方法循环解析出`testDigits`目录下的所有测试样本文件，分别调用分类器函数`classifier()`得到每个测试样本的预测结果，将预测结果和实际答案打印输出。 

程序还统计了预测错误的总数，最后通过错误总数除以测试数据总数得到预测错误率。 

整个案例的Python3代码实现在`recongnition.py`中，可以从[这里下载](https://github.com/longyg/Machine-Learning-Practice/blob/master/kNN/%E6%89%8B%E5%86%99%E8%AF%86%E5%88%AB/recongnition.py)。 

运行程序可以得到类似如下输出：

```python
分类器预测结果: 0, 实际答案是: 0
分类器预测结果: 0, 实际答案是: 0
分类器预测结果: 0, 实际答案是: 0
分类器预测结果: 0, 实际答案是: 0
分类器预测结果: 0, 实际答案是: 0
...
分类器预测结果: 9, 实际答案是: 9
分类器预测结果: 9, 实际答案是: 9
分类器预测结果: 9, 实际答案是: 9
分类器预测结果: 9, 实际答案是: 9

预测错误数: 10

错误率为: 0.010571
```

可以看到大约900个测试样本，只有10个预测错误，错误率为`1%`，这是一个不错的结果。

### k-近邻算法总结

从前面的案例可以看出：

1.  `k-近邻`算法执行效率较低。因为算法对于每个测试数据，都要计算其与所有训练样本数据的距离。比如训练数据有2000个，测试数据有900个，就要执行`2000x900`次距离公式的运算。而如果每个样本有`1024`个维度（即特征），那么每次距离公式的运算都要进行1024维的运算，这将是不小的运算量。
2.  `k-近邻`算法的开销较大。因为程序需要保存所有训练样本数据，如果训练数据集很大的话，将需要大量的存储空间。