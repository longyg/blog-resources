---
title: 《机器学习实战》读书笔记之三：决策树
cover: true
top: true
mathjax: true
date: 2018-08-05 15:49:17
group: ml
permalink: mlp-3-decision-tree
categories: 机器学习
tags:
- 机器学习
- 算法
- 决策树
keywords:
- 决策树
- 机器学习决策树算法
summary: 本文将详细介绍如何根据训练数据构造决策树，然后通过决策树创建分类器，最后通过实际案例测试分类器的预测效果。
---

### 一、概述

`决策树`算法用于解决分类问题。它的基本原理是，通过给定原始数据（即训练数据）构造一棵决策树，然后通过构造的决策树构建一个分类器，从而对未知数据进行分类。 该算法的关键是如何构造一棵合适的决策树，所谓合适，就是指找到一棵`最优的决策树`，从而使数据经过尽量少的分支最终得到数据的所属分类。 本文将详细介绍如何根据训练数据构造决策树，然后通过决策树创建分类器，最后通过实际案例测试分类器的预测效果。

### 二、构造决策树

#### 1\. 什么是决策树

何为`决策树`，我们通过实际例子可以很轻松理解。 

如下就是一棵决策树，用于判断某种海洋生物是否为鱼类： 

[![](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftytsxc52lj20h507k3z0.jpg)](http://wx3.sinaimg.cn/mw690/bd7db87egy1ftytsxc52lj20h507k3z0.jpg) 

从图中可以看到，该决策树通过两个属性来判断海洋生物是否为鱼类，两个属性分别为:

-   不浮出水面是否可以生存
-   是否有脚蹼

如果该生物不浮出水面不能生存，那该生物为非鱼类。如果不浮出水面可以生存，而且有脚蹼，那就是鱼类，否则为非鱼类。 

其中每个属性都只有两种可能的取值，因此每个属性框向下都只有两个分支。 

所以，不管决策树多么复杂，决策树始终由三个基本模块组成，：

-   `判断模块`：代表数据的属性或特征
-   `终止模块`：最后的叶子节点，表示数据经过决策树后的最终分类
-   `分支`：从一个判断模块到达下一个模块的分叉，可以是另一个判断模块或终止模块。属性有多少种取值就会有多少个分支。

大家可能想到了，上面的决策树并不是判断海洋生物是否为鱼类的唯一决策树，我们可以调换属性判断顺序，上面的决策树是先判断属性“不浮出水面是否可以生存”，如果我们改为先判断属性“是否有脚蹼”，将得到一个完全不同的决策树，如下： 

[![](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftytswws1wj20h408mjrs.jpg)](http://wx4.sinaimg.cn/mw690/bd7db87egy1ftytswws1wj20h408mjrs.jpg) 

那么究竟在实际使用中，我们应该使用哪一种决策树呢？实际上这就是机器学习决策树算法要解决的核心问题：`构造最优决策树`。 

接下来我们将介绍如何构造最优决策树。

#### 2\. 构造最优决策树

应用机器学习的决策树算法，我们需要收集一定量的训练数据，通过训练数据构造`最优决策树`。 

如何构造最优决策树？ 

核心就是从训练数据找出`划分数据集的最好特征`，然后划分数据集为多个子数据集。再递归地从子数据集找出划分子数据集的最好特征，划分子数据集，直到所有子数据集中的数据具有相同分类。 

因此为了构造最优决策树，实际上我们需要解决一个关键的算法问题：

-   找出划分数据集的最好特征的算法

接下来我们将实现这个关键算法，然后实现划分数据集，最后实现构造`最优决策树`。 

我们依然以前面的判断海洋生物是否为鱼类作为例子，假设我们有一组训练数据如下：

| 编号 | 不浮出水面是否可以生存 | 是否有脚蹼 | 是否为鱼类 |
|------|-------------|-------|-------|
| 1    | 可以生存        | 有     | 鱼类    |
| 2    | 可以生存        | 有     | 鱼类    |
| 3    | 可以生存        | 没有    | 非鱼类   |
| 4    | 不能生存        | 有     | 非鱼类   |
| 5    | 不能生存        | 有     | 非鱼类   |

通过上面给定的训练数据集，我们需要确定是以第一个特征优先分类，还是以第二个特征优先分类。 

**那么如何确定呢？** 

这里将要采用信息论理论：`香农熵`。就是可以将信息进行量化的理论，通过计算信息的熵来获知信息量。 

计算`香农熵`的公式为： 

$$H = -\sum_{i=1}^{n}p(x_i)log_2p(x_i)$$ 

其中$p(x_i)$表示数据集中第\\(x\_i\\)种分类的概率。 

至于这个公式是怎么来的，这个大家不用去关心，其实我也没有去深入了解过。他是大牛克劳德-香农发明的，也是世人所公认的。 

它用于度量数据集的混乱度，`香农熵`越大，混乱度越高。而`混乱度越高`，说明数据集包含的`信息量越大`。反之，如果某个数据集`混乱度越低`（即数据集的分类更明朗），那它所承载的`信息量越小`。 

而我们的决策树的目的是要把混乱的数据集进行分类，最终让数据集变得分类明确。也就说，要尽量让分类后的数据集的香农熵小。 

以前面的数据集为例，我们来根据香农熵公式计算一下该数据集的香农熵。 

数据集包含两种分类：`鱼类`和`非鱼类`。其中: 

- `鱼类`有两条数据，概率为： 

$$p(鱼类) = 2 / 5 = 0.4$$ 

- `非鱼类`有3条数据，概率为： 

$$p(非鱼类) = 3 / 5 = 0.6$$ 

代入香农熵公式： 

$$H = - (p(鱼类)log\_2p(鱼类) + (p(非鱼类)log\_2p(非鱼类))$$ 

$$= - (0.4 * log_20.4 + 0.6 * log_20.6)$$ 

$$= -(-0.5287712379549449-0.44217935649972373)$$ 

$$= 0.9709505944546686$$ 

接下来，用Python3实现计算数据集的香农熵。创建`decision_tree.py`文件，定义计算香农熵的`calcShannonEnt()`函数：

```python
import math

# 计算香农熵
import math
 
# 计算香农熵
def calcShannonEnt(dataSet):
    # 获取数据集数据总个数
    numEntries = len(dataSet)
    labelCounts = {}
    for featVec in dataSet:
        # 获取数据的分类，数据集的最后一个数据即是该数据的分类
        currentLabel = featVec[-1]
        if currentLabel not in labelCounts.keys():
            labelCounts[currentLabel] = 0
        labelCounts[currentLabel] += 1
    shannonEnt = 0.0
    for key in labelCounts:
        # 计算分类的概率
        prob = float(labelCounts[key]) / numEntries
        # 使用香农熵公式计算香农熵
        shannonEnt -= prob * math.log(prob, 2)
    return shannonEnt
```

然后创建前面示例的数据集：

```python
def createDataSet():
    dataSet = [['可以生存', '有', "鱼类"],
                ['可以生存', '有', "鱼类"],
                ['可以生存', '没有', "非鱼类"],
                ['不能生存', '有', "非鱼类"],
                ['不能生存', '有', "非鱼类"]]
    labels = ['不浮出水面是否可以生存', '是否有脚蹼']
    return dataSet, labels
```

```python
dataSet, labels = createDataSet()
print(calcShannonEnt(dataSet))
```
```
0.9709505944546686
```

可见，Python实现的函数的计算结果与前面的计算结果一致。 

我们也可以看到，`香农熵`的大小取决于两个因素：

1.  数据分类种类个数
2.  每种分类的概率大小。

数据`分类种类越多`，表示数据`混乱度越高`，因此`香农熵越大`。 

数据`分类越均匀`，`香农熵越大`。比如上面的例子，如果两种分类的其中每种分类的概率一样，都为`0.5`，香农熵将最大为`1.0`。 

为了确定用于划分数据集的最好特征，我们可以计算划分前与划分后的数据集的`信息增益`。所谓信息增益，就是划分前数据集的香农熵与划分后数据集的香农熵的`差值`： 

$$I = H_{划分前} - H_{划分后}$$ 

差值越大，信息增益越大。如果某个特征划分数据集后信息增益最大，那么该特征就是我们要寻找的划分数据集的`最佳特征`。 

下面我们来计算前面示例的两个特征分别划分数据集后的信息增益，然后比较信息增益大小，`信息增益最大的特征`，即为划分数据集的`最好特征`。 

按第一个特征“不浮出水面是否可以生存”划分数据集，结果如下： 

[![](http://wx3.sinaimg.cn/large/bd7db87egy1fy2m5535vyj212z0is0uo.jpg)](http://wx3.sinaimg.cn/large/bd7db87egy1fy2m5535vyj212z0is0uo.jpg) 

信息增益为： 

$$I_1 = H_{划分前} - H_{划分后}$$

$$= 0.9709505944546686 - 0.5509775004326937$$ 

$$= 0.4199730940219749$$ 

按第二个特征“是否有脚蹼”划分数据集，结果如下： 

[![](http://wx1.sinaimg.cn/large/bd7db87egy1fy2m560278j212i0is75z.jpg)](http://wx1.sinaimg.cn/large/bd7db87egy1fy2m560278j212i0is75z.jpg) 

信息增益为： 

$$I_2 = H_{划分前} - H_{划分后}$$ 

$$= 0.9709505944546686 - 0.8$$ 

$$= 0.1709505944546686$$ 

比较按两种特征划分后的信息增益，由于$I_1 > I_2$，因此，第一个特征“不浮出水面是否可以生存”为划分该数据集的`最好特征`。 

现在，相信你已经了解了通过计算`信息增益`来寻找划分数据集的`最好特征`的算法原理。接下来，我们用Python3实现该算法。 

首先定义`splitDataSet()`函数用于划分数据集：

```python
# 划分数据集
# dataSet： 将被进行划分的数据集
# axis: 特征索引，表示按第几个特征进行划分，从0开始
# value: 特征的值，将把第axis个特征具有相同value的数据划分为同一个子集
def splitDataSet(dataSet, axis, value):
    retDataSet = []
    for featVec in dataSet:
        if featVec[axis] == value:
            # 去掉用于划分的特征，只保留还未用于划分的特征
            reducedFeatVec = featVec[:axis]
            reducedFeatVec.extend(featVec[axis + 1:])
            retDataSet.append(reducedFeatVec)
    return retDataSet
```

接下来，定义`chooseBestFeatureToSplit()`函数，实现寻找划分数据集的最好特征：

```python
# 选择划分数据集的最好特征
def chooseBestFeatureToSplit(dataSet):
    numFeatures = len(dataSet[0]) - 1
    # 计算划分前的香农熵
    baseEntropy = calcShannonEnt(dataSet)
    bestInfoGain = 0.0
    bestFeature = -1
    # 循环用每个特征对数据集进行划分
    for i in range(numFeatures):
        featList = [example[i] for example in dataSet]
        uniqueVals = set(featList)
        newEntropy = 0.0
        # 循环每个特征的所有取值，进行划分
        for value in uniqueVals:
            # 划分数据集，获得划分后的子集
            subDataSet = splitDataSet(dataSet, i, value)
            # 计算子集的数据比例
            prob = len(subDataSet) / float(len(dataSet))
            # 计算按特征i进行划分后，所有子集的总香农熵
            newEntropy += prob * calcShannonEnt(subDataSet)
        # 计算信息增益
        infoGain = baseEntropy - newEntropy
        # 比较信息增益，将信息增益大的特征赋予bestFeature变量
        if (infoGain > bestInfoGain):
            bestInfoGain = infoGain
            bestFeature = i
    return bestFeature
```

测试一下：

```python
bestFeature = chooseBestFeatureToSplit(dataSet)
print(bestFeature)
```

输出：

```
 0
```

输出为`0`，表示第一个特征“不浮出水面是否可以生存”，与我们在前面的推导结果是一致的。 

现在，我们已经实现了决策树的核心算法：`找出划分数据集的最好特征`。接下来，我们将使用该算法构造决策树。 

`构造决策树`的基本步骤为：

1.  寻找划分数据集的`最好特征`
2.  划分数据集
3.  创建分支节点
4.  循环每个子集，递归地重复步骤1-3，如果子集包含的数据都属于同一分类，则返回。

但有一种情况，如果所有特征都被用于划分后，依然有某个子集的数据不完全属于同一分类，这种情况下，我们将直接返回该子集中出现次数最多的分类。 

因此，我们首先实现`majorityCnt()`函数用于获取出现次数最多的分类：

```python
import operator
 
# 返回分类列表中出现次数最多的分类
def majorityCnt(classList):
    classCount = {}
    for vote in classList:
        if vote not in classCount.keys():
            classCount[vote] = 0
        classCount[vote] += 1
    sortedClassCount = sorted(classCount.items(), key=operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]
```

定义`createTree()`函数创建决策树：

```python
# 创建决策树
def createTree(dataSet, labels):
    # 获取所有数据的分类，每条数据的最后一列为该数据的分类，因此可以通过索引-1获取。
    classList = [example[-1] for example in dataSet]
    # 如果分类列表中所有分类都相同，则直接返回
    if classList.count(classList[0]) == len(classList):
        return classList[0]
    # 如果分类列表中分类不一样，且数据集中每条数据都只有一项数据了，
    # 这表示数据集中不包含特征数据了，也就是说所有特征都已经被用于划分数据集了。
    # 这种情况下，返回出现次数最多的分类
    if len(dataSet[0]) == 1:
        return majorityCnt(classList)
    # 找出用于划分数据集的最好特征的索引
    bestFeature = chooseBestFeatureToSplit(dataSet)
    # 划分数据集的最好特征的名称
    bestFeatureLabel = labels[bestFeature]
    # 以最好特征名称为key，初始化决策树
    myTree = {bestFeatureLabel:{}}
    # 找到最好特征后，将其从特征列表中删除，以免后续重复使用特征进行分类。
    del(labels[bestFeature])
    # 获取最好特征所有的值
    featureValues = [example[bestFeature] for example in dataSet]
    # 去重
    uniqueVals = set(featureValues)
    # 循环特征的值
    for value in uniqueVals:
        subLabels = labels[:]
        # 用前面找出的最好特征与特征值划分数据集，再递归地对子集构建决策树
        myTree[bestFeatureLabel][value] = createTree(splitDataSet(dataSet, bestFeature, value), subLabels)
    return myTree
```
测试 一下：

```python
tree = createTree(dataSet, labels)
print(tree)
```

输出为：

```
{'不浮出水面是否可以生存': {'可以生存': {'是否有脚蹼': {'有': '鱼类', '没有': '非鱼类'}}, '不能生存': '非鱼类'}}
```

输出为一个嵌套的字典，第一层字典的`key`为数据集的`最好特征`，值为`分支子树`。子树为多个字典，对应最好特征的多个值。

### 三、创建决策树分类器

到目前为止，我们已经构造出`最优决策树`。下一步，我们将使用决策树构建`分类器`，从而可以使用分类器对未知数据进行分类。 

定义`classify()`函数：

```python
# 决策树分类器
# inputTree: 决策树
# featLabels: 特征标签列表
# testVec: 测试向量
def classify(inputTree, featLabels, testVec):
    # 获取决策树的第一个key，就是第一个分类特征
    firstStr = list(inputTree.keys())[0]
    # 获取第一个分类特征的值，也就是其分支
    secondDict = inputTree[firstStr]
    # 获取第一个特征的在特征标签列表中的索引
    featIndex = featLabels.index(firstStr)
    # 循环第一个特征的分支树
    for key in secondDict.keys():
        if testVec[featIndex] == key:
            # 如果分支是一个字典，说明还有包含子判断模块的子树
            if type(secondDict[key]).__name__ == 'dict':
                # 递归调用分类器，进入子树判断分类
                classLabel = classify(secondDict[key], featLabels, testVec)
            else:
                # 否则说明已经找到最终分类，直接返回。
                classLabel = secondDict[key]
    return classLabel
```
那么我们来测试一下分类器：

```python
result = classify(tree, labels, ["不能生存", "有"])
print(result)
```

输出为：

```
非鱼类
```

给定特征数据，不浮出水面不能生存，有脚蹼，决策树分类器预测结果为非鱼类。 

有了这个分类器，我们可以对任意输入的数据进行预测。当然，对于这个预测海洋生物是否为鱼类的例子，实在是太简单了，因为只有两个特征，每个特征都只有两种取值，因此总的组合数据也就只有几条。完全没必要这么费力的创建决策树然后创建分类器。但是，因为简单，我们用这个例子来解释决策树算法的细节就更容易被理解。 

现实中，我们面对的数据集会比这个例子复杂得多，比如可能有成千上万个特征，每个特征都有很多取值。但是无论如何复杂，构造决策树的原理都是完全不变的。并且，我们实现的Python3代码是适用于任何复杂数据集的。

### 四、决策树应用案例 - 预测隐形眼镜类型

本案例是通过决策树预测患者需要佩戴什么类型的隐形眼镜。 

我们有一份眼镜类型的数据集文件：`lenses.txt`，请[点击链接](https://github.com/longyg/Machine-Learning-Practice/blob/master/DecisionTree/lenses.txt)下载数据文件。 

包含4个特征，分别为`age`, `prescript`, `astigmatic`, `tearRate`。有些特征有两个取值，有些有三个取值。 

我们首先将数据文件解析为`dataSet`格式，然后使用第二节中实现的函数创建决策树和分类器。创建`lenses.py`文件：

```python
import decision_tree
 
if __name__ == '__main__':
    fr = open('lenses.txt')
    # 读取数据文件的每一行，然后以\t分割成列表
    lenses = [inst.strip().split('\t') for inst in fr.readlines()]
    lensesLabels = ["age", "prescript", "astigmatic", "tearRate"]
    # 使用decision_tree实现的createTree()函数创建决策树
    lensesTree = decision_tree.createTree(lenses, lensesLabels)
    print(lensesTree)
    
    # 注意，我们在使用分类器时，要重新传入分类标签列表，不能重用前面的分类标签列表。因为在创建决策树函数中，会删除标签列表里的数据。
    labels = ["age", "prescript", "astigmatic", "tearRate"]
    # 使用分类器函数预测未知数据
    result = decision_tree.classify(lensesTree, labels, ["young", "hyper", "yes", "reduced"])
    print(result)
```

执行代码输出如下：

```
{'tearRate': {'normal': {'astigmatic': {'yes': {'prescript': {'hyper': {'age': {'young': 'hard', 'pre': 'no lenses', 'presbyopic': 'no lenses'}}, 'myope': 'hard'}}, 'no': {'age': {'young': 'soft', 'pre': 'soft', 'presbyopic': {'prescript': {'hyper': 'soft', 'myope': 'no lenses'}}}}}}, 'reduced': 'no lenses'}}
no lenses
```

我们使用前面实现的`createTree()`函数构建了决策树，然后将其传入`classify()`函数用于预测未知隐形眼镜的类型。 

也可以看到，这里创建的`决策树`相对前面的决策树要更复杂一些，我们可以将其用决策树图展示出来看看： 

[![](http://wx4.sinaimg.cn/large/bd7db87egy1fy2m56n1jhj20jj0d8gms.jpg)](http://wx4.sinaimg.cn/large/bd7db87egy1fy2m56n1jhj20jj0d8gms.jpg)

### 五、小结

本文所有代码放在github，[点击此处](https://github.com/longyg/Machine-Learning-Practice/tree/master/DecisionTree)查看完整代码。 

`决策树算法`是一个非常简单，而且非常容易理解的算法。本文详细讲解了构建决策树的算法原理，对如何找出划分数据集的最好特征进行了推导，使用的是`ID3算法`（即通过计算数据集的`香农熵`从而计算出`信息增益`，通过比较信息增益来确定`最佳特征`），非常容易理解。当然实现决策树还有其他算法，后续继续讲解。
