---
title: 【TensorFlow 2.0教程】初学者入门指南
cover: false
top: false
date: 2019-06-02 22:12:50
group: ml
permalink: tf2-get-started-for-beginners
categories: TensorFlow
tags:
- 机器学习
- TensorFlow
keywords:
- TensorFlow 2 教程
summary: 本文是TensorFlow 2.0入门示例
---


本文是`TensorFlow 2.0`入门示例，使用`TensorFlow 2.0`对MNIST手写数字进行识别，从而展示了基于`TensorFlow 2.0`进行开发的最简单的流程。 

首先导入`TensorFlow 2.0`

```python
import tensorflow as tf

print(tf.__version__)
```
```
2.0.0-alpha0
```
加载并准备[MNIST手写数字数据集](http://yann.lecun.com/exdb/mnist/)，并将样本从整数转换为浮点数:

```python
mnist = tf.keras.datasets.mnist
 
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train/255.0, x_test/255.0
 
print(x_train.shape)
print(x_test.shape)
```

```
Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/mnist.npz
11493376/11490434 [==============================] - 0s 0us/step
(60000, 28, 28)
(10000, 28, 28)
```

首次运行时会下载数据集，从输出的log中可以看到。当已经下载后，不会再重复下载。 

该数据集中有`60000`个训练样本，`10000`个测试样本，每个样本都是`28x28`的二维数组，数组中每个数字代表图片的一个像素值。由于像素值最大为`255`，为了对数据归一化，我们对每个值除以`255`。

接下来，构建一个最基本的层叠模型（`Sequential`），并选择一个`优化器`和`损失函数`进行训练:

```python
model = tf.keras.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation='softmax')
])
 
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
```

通过`tf.keras.Sequential`创建模型后，调用模型的`compile`函数编译模型，编译时指定优化器，损失函数，和监测的指标，这里只监测了准确率（`Accuracy`）。 

模型创建并编译后，开始训练模型：

```python
model.fit(x_train, y_train, epochs=5)
```

```
Epoch 1/5
60000/60000 [==============================] - 3s 48us/sample - loss: 0.2970 - accuracy: 0.9124
Epoch 2/5
60000/60000 [==============================] - 3s 45us/sample - loss: 0.1433 - accuracy: 0.9577
Epoch 3/5
60000/60000 [==============================] - 3s 45us/sample - loss: 0.1077 - accuracy: 0.9673
Epoch 4/5
60000/60000 [==============================] - 3s 44us/sample - loss: 0.0881 - accuracy: 0.9732
Epoch 5/5
60000/60000 [==============================] - 3s 45us/sample - loss: 0.0752 - accuracy: 0.9770
 
<tensorflow.python.keras.callbacks.History at 0xf8a2b00>
```

训练模型使用模型的`fit`函数，传入训练样本数据，并指定训练迭代次数，这里只迭代了`5`次，即对所有训练样本重复进行了`5`次训练。 

训练完成后，我们可以使用测试数据对模型进行评估：

```python
model.evaluate(x_test, y_test)
```

```
10000/10000 [==============================] - 0s 28us/sample - loss: 0.0756 - accuracy: 0.9779
[0.07561568637117744, 0.9779]
```

可以看到训练好的模型，对于测试数据达到了接近`98%`的准确率。 

以上就是使用`TensorFlow 2.0`最简单的入门示例，也展示了使用`TensorFlow 2.0`进行开发的基本流程。`TensorFlow 2.0`使用了`keras`作为高阶API，相对于`TensorFlow 1.x`在编码以及开发效率上简化了很多。 

本文完整源代码请参考[这里](https://github.com/longyg/machine-learning/blob/master/tensorflow-2.0/1-get%20started.ipynb)。
