---
title: 《机器学习实战》读书笔记之一：k-近邻算法原理与代码实现详解
cover: true
top: true
mathjax: true
date: 2018-07-21 22:11:01
group: ml
permalink: mlp-1-knn
categories: 机器学习
tags:
- 机器学习
- 算法
keywords:
- 机器学习KNN算法
summary: 介绍《机器学习实战》书中讲解的第一个机器学习算法：k-近邻算法，也称kNN（k-NearestNeighbor的缩写）算法
---


### 一、前言

本系列文章是学习《机器学习实战》一书的读书笔记，并非按原文照抄，而是在理解原书的基础上融入了本人个人理解。同时，原书代码是基于Python2实现的，而本系列文章所有代码是用Python3实现的，因此代码与原书也会稍有不同。如果你使用本系列文章的代码，请一定在Python3下运行，否则你会遇到意想不到的问题。 

此文是《机器学习实战》的读书笔记的第一篇，介绍该书中讲解的第一个机器学习算法：`k-近邻算法`，也称`kNN`（k-NearestNeighbor的缩写）算法。

### 二、算法原理

将输入数据的每个特征数据与训练样本集对应的特征数据进行比较，从而提取出样本集中特征最相似（即最邻近）的k个数据，将这k个数据中比例最高的分类标签作为输入数据的分类标签。 由于每个训练样本数据都是有标签的，所以`kNN`算法是监督学习的一种算法。

#### 1\. 举例说明

假设有一组电影分类的样本数据集，根据电影中打斗镜头数和接吻镜头数的不同，被区分为动作片和爱情片。 

有如下7部电影样本数据，其中6部已知类型，1部未知类型。我们希望从这6部电影中找到某种规律，从而预测未知类型的“电影7”属于什么类型。

| 电影名称 | 打斗镜头数 | 接吻镜头数 | 电影类型 |
|------|-------|-------|------|
| 电影1  | 3     | 104   | 爱情片  |
| 电影2  | 2     | 100   | 爱情片  |
| 电影3  | 1     | 81    | 爱情片  |
| 电影4  | 101   | 10    | 动作片  |
| 电影5  | 99    | 5     | 动作片  |
| 电影6  | 98    | 2     | 动作片  |
| 电影7  | 18    | 90    | ？    |


将上面样本数据在图中表示出来： 

完整代码在[这里](https://github.com/longyg/Machine-Learning-Practice/blob/master/kNN/showDataSet.py)。

```python
import numpy as np
import matplotlib.pyplot as plt
 
x=np.array([3,2,1,101,99,98,18])
y=np.array([104,100,81,10,5,2,90])
 
colors = (['black','black','black','black','black','black','red'])
plt.scatter(x, y, c=colors)
 
plt.xlabel('打斗镜头数')
plt.ylabel("接吻镜头数")
plt.rcParams['font.sans-serif'] = ['FangSong']
plt.rcParams['axes.unicode_minus'] = False
plt.show()
```

[![](http://wx2.sinaimg.cn/mw690/bd7db87egy1fsvwj6chebj20hs0d50st.jpg)](http://wx2.sinaimg.cn/mw690/bd7db87egy1fsvwj6chebj20hs0d50st.jpg) 

图中6个黑色的点表示上面样本数据集中的6部已知类型的电影，其中3部爱情片，3部动作片。红色的点表示我们将要预测的未知类型的“电影7”。 

根据`kNN`算法原理，我们需要首先计算出图中红色点与所有黑色点的距离。大家都还记得计算两个点之间的距离公式吧。 如果点1记为（$x_1$，$y_1$），点2记为（$x_2$，$y_2$），那么点1与点2的距离为： 

$$\sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^{2}}$$ 

通过距离公式计算得到所有6个黑色点到红色点的距离如下表：


| 电影名称 | 与电影7的距离计算公式                                             | 与电影7的距离 |
|------|---------------------------------------------------------|---------|
| 电影1  | $\sqrt{(3-18)^{2} + (104-90)^{2}}$  | 20.5   |
| 电影2  | $\sqrt{(2-18)^{2} + (100-90)^{2}}$  | 18.7   |
| 电影3  | $\sqrt{(1-18)^{2} + (81-90)^{2}}$   | 19.2   |
| 电影4  | $\sqrt{(101-18)^{2} + (10-90)^{2}}$ | 115.3  |
| 电影5  | $\sqrt{(99-18)^{2} + (5-90)^{2}}$   | 117.4  |
| 电影6  | $\sqrt{(98-18)^{2} + (2-90)^{2}}$   | 118.9  |


按照距离递增排序，可以找到k个距离最近的电影。 假设`k=3`，那么距离最近的3部电影是`电影2`，`电影3`和`电影1`。从输入数据集中知道，这三部电影都是爱情片，因此我们判定`电影7`是爱情片。 注意，`kNN`算法是按照距离最近的`k`个数据的类型来决定未知数据类型的。如果`k`个距离最近的数据不是唯一类型，将把`k`个数据中比例最高的类型作为未知数据的类型。 例如，如果我们把k设为4，得到距离最近的4部电影分别是`电影2`，`电影3`，`电影1`和`电影4`。这4部电影中，其中有三部爱情片，比例是75%，只有一部是动作片，比例是25%。因此我们判定未知电影还是爱情片。

### 三、算法实现

根据前面的算法原理以及例子，我们将k-近邻算法用Python实现。实现代码中主要进行以下操作：

1.  计算已知类别数据集中每个点与未知点的距离
2.  按照距离递增排序
3.  选取与未知点距离最小的k个点
4.  确定前k个点中所有类别的比例，
5.  返回前k个点比例最高的类别，作为未知点的预测分类。

#### 1\. Python3代码实现

创建`knn.py`文件并输入如下代码：

```python
from numpy import *
import operator
import sys
 
def classifier(inX, dataSet, labels, k):
    dataSetSize = dataSet.shape[0]
    diffMat = tile(inX, (dataSetSize, 1)) - dataSet
    sqDiffMat = diffMat ** 2
    sqDistances = sqDiffMat.sum(axis = 1)
    distances = sqDistances ** 0.5
    sortedDistIndicies = distances.argsort()
    classCount = {}
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1
    sortedClassCount = sorted(classCount.items(), key = operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]
 
def createDataSet():
    dataSet = array([[3,104],[2,100],[1,81],[101,10],[99,5],[98,2]])
    labels = ['爱情片', '爱情片', '爱情片', '动作片', '动作片', '动作片']
    return dataSet, labels
 
if __name__ == '__main__':
    dataSet, labels = createDataSet()
    input = [int(sys.argv[1]), int(sys.argv[2])]
    k = 4
    output = classifier(input, dataSet, labels, k)
    print(output)
```

#### 2\. 运行代码

在命令行当前目录下执行上面python脚本，并将未知电影的打斗镜头数和接吻镜头数作为参数传递给脚本，结果将输出如下：

```bash
C:\tmp> knn.py 18 90
爱情片
```

可见，上面的代码输出未知电影为“爱情片”，结果与我们之前推导的结果完全一致。 现在我们可以用这个程序来预测任意的输入数据了，例如：

```bash
C:\tmp> knn.py 10 100
爱情片

C:\tmp> knn.py 90 10
动作片
```

从[这里下载](https://github.com/longyg/Machine-Learning-Practice/blob/master/kNN/knn.py)`knn.py`的完整代码。

#### 3\. 代码解析

程序首先导入依赖模块，本算法将用到最重要的`numpy`，以及`operator`和`sys`模块，因此将他们导入。

```python
from numpy import *
import operator
import sys
```

上面的核心代码在分类器函数`classifier()`中，有4个输入参数，分别为：

-   `inX`: 输入数据，将要预测的数据，用数组表示，如[18, 90]
-   `dataSet`: 训练数据集，即已知类型的数据，用numpy的矩阵表示
-   `labels`: 训练数据集的分类，用数组表示，数组中每一个值表示`dataSet`中对应的数据的分类。
-   `k`：k-近邻算法的k值

`classifier()`函数第一行获取`dataSet`数据集的数据个数，使用了numpy数组的`shape`属性。`dataSet`是传入的参数，它是由`createDataSet()`函数创建的numpy数组（二维矩阵）:

```python
dataSet = array([[3,104], [2,100], [1,81], [101,10], [99,5], [98,2]])
print(dataSet)
```

```bash
[[  3 104]
 [  2 100]
 [  1  81]
 [101  10]
 [ 99   5]
 [ 98   2]]
```


numpy的`array()`函数可以通过传入一个python标准的list来创建一个numpy数组，numpy数组可以是多维的，对于二维数组也可称为矩阵。而我们的输入数据集`dataSet`就是一个二维数组，即是一个6x2的矩阵，6表示总共有6个数据样本，2表示每个数据样本有2列，即2个特征（打斗镜头数和接吻镜头数）。所以我们的数据集`dataSet`是一个6行2列的矩阵。

`dataSet.shape`是numpy提供的，可以获取numpy数组的维数，因为dataSet是一个6行2列的矩阵，因此dataSet.shape输出如下：

```python
dataSet.shape
```

```bash
(6, 2)
```

而`classifier()`函数第一行调用`dataSet.shape[0]`，表示返回第一维的大小：

```python
dataSetSize = dataSet.shape[0]
print(dataSetSize)
```
```bash
6
```

`classifier()`函数第二行使用了numpy的`tile()`函数，该函数原型为`numpy.tile(A,reps)`，接收两个参数，表示把A根据reps重复输出。请参见[numpy.tile](https://docs.scipy.org/doc/numpy-1.13.0/reference/generated/numpy.tile.html)对该函数的详细介绍。 

`classifier()`函数中先按如下调用`tile()`函数：

```python
tile(inX, (dataSetSize, 1))
```

第一个参数是由命令行输入的参数构成的Python标准list:

```python
input = [int(sys.argv[1]), int(sys.argv[2])]
```

假如输入18, 90，则input为：

```python
inX = [18, 91]
print(inX)
```
```bash
[18, 91]
```

我们先看看`tile(inX, (dataSetSize, 1))`输出长什么样子：

```python
tmpMat = tile(inX, (dataSetSize, 1))
print(tmpMat)
```

```bash
[[18 91]
 [18 91]
 [18 91]
 [18 91]
 [18 91]
 [18 91]]
```

可以看到，`tile()`函数将输入的list在行方向上重复了6次，列方向上1次，从而变成了一个6行2列的矩阵。 

接下来，用新构造的矩阵与数据集`dataSet`做减法。矩阵的减法大家都还记得吧，就是对应位置的数值做减法，因此得到相减之后的矩阵`diffMat`如下。
```python
diffMat = tmpMat - dataSet
print(diffMat)
```
```bash
[[ 15 -13]
 [ 16  -9]
 [ 17  10]
 [-83  81]
 [-81  86]
 [-80  89]]
```

由于矩阵减法要求两个矩阵具有相同维数，因此我们也可以理解为什么需要先用`tile()`函数构造一个与`dataSet`相同维数的矩阵了吧。 接下来，对`diffMat`做平方运算，也就是对`diffMat`矩阵的每一个数做平方运算，得到`sqDiffMat`如下：

```python
sqDiffMat = diffMat**2
print(sqDiffMat)
```
```bash
[[ 225  169]
 [ 256   81]
 [ 289  100]
 [6889 6561]
 [6561 7396]
 [6400 7921]]
```

接下来，对`sqDiffMat`矩阵求和。这里使用了numpy的`sum()`函数，请参见[numpy.sum](https://docs.scipy.org/doc/numpy/reference/generated/numpy.sum.html)对该函数的详细介绍。 

`sum()`函数的`axis`参数指定对哪一维数据求和，如果不指定`axis`，将对整个矩阵所有数据求和。`axis=0`表示对矩阵的第一维数据求和，对于上面的二维矩阵，就是对每一列求和。`axis=1`表示将矩阵的第二维数据求和，对于上面的二维矩阵，也就是对每一行求和，比如第一行的两个数相加得到第一个值，225 + 169 = 394，以此类推对所有行求和，最后得到一个数组如下：

```python
sqDistances = sqDiffMat.sum(axis=1)
print(sqDistances)
```
```bash
[  394   337   389 13450 13957 14321]
```

接下来对求和后的数组`sqDistances`求平方根，得到所有点与未知点的距离：

```python
distances = sqDistances**0.5
print(distances)
```
```bash
[ 19.84943324  18.35755975  19.72308292 115.97413505 118.13974776 119.67038063]
```

至此，我们已经求出了数据集`dataSet`中所有数据与输入数据之间的距离，`distances`数组中的每一个数就是`dataSet`中每一组数据与输入数据的距离。 

可以看到，我们使用numpy的数组操作，代码非常简单，一次性可以求出所有数据到输入数据的距离，而不用通过遍历来求每个数据到输入数据的距离。 

接下来，对上面求出来的距离进行排序。首先使用numpy的`argsort()`函数对distances排序，这个函数返回数组数值从小到大的索引值：

```python
sortedDistIndicies = distances.argsort()
print(sortedDistIndicies)
```
```bash
[1 2 0 3 4 5]
```

从上面输出可以看出，索引为1（也就是数组的第2个，因为索引是从0开始的，）的数值最小，排在第一位，我们回头去看看distances数组，确实是数组的第2个数最小，为`18.35755975`。索引为2（数组的第3个数）的数值倒数第二小，排在第二位，数值为`19.72308292`。以此类推，可以得到上面相同的输出，从而也证明了`argsort()`函数的正确性。 

接下来，程序通过for循环来找出距离最近的前k个数的类别，并统计每种类别的个数。 

本程序中，k被设置为固定值4，所以程序只需要循环4次对前面已经排好序的数组进行统计。而labels是`createDataSet()`函数设置的，对应着dataSet中每组数组的类型。

```python
k = 4
labels = ['爱情片', '爱情片', '爱情片', '动作片', '动作片', '动作片']
```

程序定义了一个集合`classCount`用于存放统计结果。每次循环先获取当前的类型，再对这个类型进行累加，最后将统计结果保存到集合classCount中。

```python
classCount = {}
for i in range(k):
    voteIlabel = labels[sortedDistIndicies[i]]
    classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1
```

我们打印`classCount`将得到如下输出：

```python
print(classCount)
```
```bash
{'爱情片': 3, '动作片': 1}
```

从输出可以看出，经过统计后，前k（也就是4）个距离最近的数据中，类型为动作片的个数为1，类型为爱情片的个数为3。 

最后，程序对`classCount`集合进行排序，按照集合value值从高到低排序，得到一个排好序的tuple列表。这里用到了Python内置的`sorted`函数。请参见[Python 内置函数sorted()在高级用法](https://www.cnblogs.com/brad1994/p/6697196.html)

```python
sortedClassCount = sorted(classCount.items(), key=operator.itemgetter(1), reverse=True) 
print(sortedClassCount) 
```
```bash
[('爱情片', 3), ('动作片', 1)]
```

最后我们只需要返回排好序的tuple列表的第一个，则得到`tuple('爱情片',3)`，再返回这个tuple的第一个，则得到程序预测的结果类型。

```python
type = sortedClassCount[0][0]
print(type)
```
```bash
爱情片
```