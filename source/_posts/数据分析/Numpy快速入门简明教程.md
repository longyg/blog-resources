---
title: Numpy快速入门简明教程
cover: false
top: false
date: 2018-08-02 14:27:52
group: data-analysis
permalink: numpy-quickstart-tutorial
categories: 数据分析
tags:
- 机器学习
- 数据分析
keywords:
- 数据处理
- Numpy数据处理
summary: 本教程是Numpy的入门教程
---


本教程是`Numpy`的入门教程，基于官方《[Quickstart tutorial](https://docs.scipy.org/doc/numpy/user/quickstart.html)》 `numpy`不是Python默认内置模块，所以在使用`numpy`之前，我们需要导入`numpy`模块：

```python
import numpy as np
```

### 一、创建`numpy`数组

#### 1\. 使用numpy.array()函数[¶](http://127.0.0.1:8889/notebooks/LearnPython/numpy.ipynb#1.-使用numpy.array()函数)

##### 1)  通过`List`创建numpy数组

传入Python标准的list

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
print(a)
```

输出：
```python
[[1 2 3]
 [4 5 6]]
```

##### 2) 通过`tuple`创建numpy数组

传入tuple

```python
b = np.array(((1, 2, 3), (4, 5, 6)))
print(b)
```

输出：

```python
[[1 2 3]
 [4 5 6]]
```

#### 2\. 使用`arange()`函数创建数组

`arange()`函数用来创建序列数组 

##### 1） 传入单个参数时，可以创建从0开始到传入参数的按1递增的序列数组

```python
c = np.arange(20)
print(c)
```

输出：

```python
[ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19]
```

##### 2） 传入2个参数，可以创建从第1个参数开始到第2个参数的按1递增的序列数组

```python
d = np.arange(10,20)
print(d)
```

输出：

```python
[10 11 12 13 14 15 16 17 18 19]
```

##### 3） 传入3个参数，可以创建从第1个参数开始到第2个参数的按第3个参数递增的序列数组

```python
e = np.arange(10, 20, 3)
print(e)
```

输出：
```
[10 13 16 19]
```

当然也可以用该方法创建一个递减序列数组：

```python
f = np.arange(20, 10, -3)
print(f)
```

输出：

```
[20 17 14 11]
```

#### 3\. 使用`linspace()`函数创建数组

`linspace()`同样可以像`arange()`函数那样创建序列数组。 在有些情况下，我们不知道递增数值是多少，而只想产生某个数值范围内的指定个数的序列数组。这种情况下， 使用`linspace()`比`arange()`函数更方便。因为`linspace()`会自动计算递增数值。 如下，生成了一个从0递增到2的包含9个元素的序列数组：

```python
e = np.linspace(0, 2, 9)
print(e)
print(e.dtype)
```

输出：
```
[0.   0.25 0.5  0.75 1.   1.25 1.5  1.75 2.  ]
float64
```
下面例子，通过`linspace()`函数生成0到2倍pi值的包含10个元素的递增序列数组，然后对生成的序列数组求sin值。

```python
from numpy import pi
e = np.linspace(0, 2*pi, 10)
print(e)
e = np.sin(e)
print(e)
```

输出：

```
[0.         0.6981317  1.3962634  2.0943951  2.7925268  3.4906585
 4.1887902  4.88692191 5.58505361 6.28318531]
[ 0.00000000e+00  6.42787610e-01  9.84807753e-01  8.66025404e-01
  3.42020143e-01 -3.42020143e-01 -8.66025404e-01 -9.84807753e-01
 -6.42787610e-01 -2.44929360e-16]
 ```