---
title: 【TensorFlow 2.0教程】对结构化数据分类
cover: false
top: false
date: 2019-06-09 20:12:53
group: ml
permalink: tf2-classify-structured-data
categories: TensorFlow
tags:
- 机器学习
- TensorFlow
keywords: 
- TensorFlow 2 教程
summary: 本教程将介绍如何对结构化数据进行分类，例如CSV中的表格数据
---

本教程将介绍如何对结构化数据进行分类，例如CSV中的表格数据。我们将使用`Keras`定义模型，并使用`feature columns` 作为桥梁，将CSV中的列映射到用于训练模型的特征。本教程将包含如下几方面的完整的代码演示：

-   使用`Pandas`加载CSV数据
-   构建一个输入管道（`pipeline`），使用`tf.data API`对数据进行`批处理`和`洗牌`。
-   使用`feature columns` API将CSV中的列映射到用来训练模型的`特征`。
-   使用`Keras`构建、训练和评估模型。

### 一、数据集介绍

我们将使用克利夫兰心脏病临床基金会提供的一个较小的[数据集](https://archive.ics.uci.edu/ml/datasets/heart+Disease)。CSV文件中有几百行，每一行描述一个病人，每一列为一个特征。我们将使用这些信息来预测患者是否患有心脏病，这是一个`二元分类`问题。 

下面是对该数据集的描述。注意，有些列是数值型的，有些列是分类列。

| Column   | Description                                                      | Feature Type   | Data Type |
|----------|------------------------------------------------------------------|----------------|-----------|
| Age      | Age in years                                                     | Numerical      | integer   |
| Sex      | (1 = male; 0 = female)                                         | Categorical    | integer   |
| CP       | Chest pain type (0, 1, 2, 3, 4)                                | Categorical    | integer   |
| Trestbpd | Resting blood pressure (in mm Hg on admission to the hospital) | Numerical      | integer   |
| Chol     | Serum cholestoral in mg/dl                                       | Numerical      | integer   |
| FBS      | (fasting blood sugar > 120 mg/dl) (1 = true; 0 = false)      | Categorical    | integer   |
| RestECG  | Resting electrocardiographic results (0, 1, 2)                 | Categorical    | integer   |
| Thalach  | Maximum heart rate achieved                                      | Numerical      | integer   |
| Exang    | Exercise induced angina (1 = yes; 0 = no)                      | Categorical    | integer   |
| Oldpeak  | ST depression induced by exercise relative to rest               | Numerical      | integer   |
| Slope    | The slope of the peak exercise ST segment                        | Numerical      | float     |
| CA       | Number of major vessels (0-3) colored by flourosopy           | Numerical      | integer   |
| Thal     | 3 = normal; 6 = fixed defect; 7 = reversable defect              | Categorical    | string    |
| Target   | Diagnosis of heart disease (1 = true; 0 = false)               | Classification | integer   |


#### 导入TensorFlow和其他库

```python
import numpy as np
import pandas as pd

import tensorflow as tf

from tensorflow import feature_column
from tensorflow.keras import layers
from sklearn.model_selection import train_test_split
```

### 二、使用Pandas导入数据集

`Pandas`是一个Python库，有许多用于加载和处理结构化数据的实用工具。我们将使用`Pandas`从一个URL下载数据集，并将其加载到`dataframe`中。

```python
URL = 'https://storage.googleapis.com/applied-dl/heart.csv'
dataframe = pd.read_csv(URL)
dataframe.head()
```

|   | age | sex | cp | trestbps | chol | fbs | restecg | thalach | exang | oldpeak | slope | ca | thal       | target |
|---|-----|-----|----|----------|------|-----|---------|---------|-------|---------|-------|----|------------|--------|
| 0 | 63  | 1   | 1  | 145      | 233  | 1   | 2       | 150     | 0     | 2.3    | 3     | 0  | fixed      | 0      |
| 1 | 67  | 1   | 4  | 160      | 286  | 0   | 2       | 108     | 1     | 1.5    | 2     | 3  | normal     | 1      |
| 2 | 67  | 1   | 4  | 120      | 229  | 0   | 2       | 129     | 1     | 2.6    | 2     | 2  | reversible | 0      |
| 3 | 37  | 1   | 3  | 130      | 250  | 0   | 0       | 187     | 0     | 3.5    | 3     | 0  | normal     | 0      |
| 4 | 41  | 0   | 2  | 130      | 204  | 0   | 2       | 172     | 0     | 1.4    | 1     | 0  | normal     | 0      |

### 三、将数据集拆分为训练、验证和测试集

我们下载的数据集在一个单一的CSV文件中，我们将把它们拆分为`训练集`，`验证集`和`测试集`。

```python
train, test = train_test_split(df, test_size=0.2)
train, val = train_test_split(train, test_size=0.2)
 
print(len(train), 'train examples')
print(len(val), 'validation examples')
print(len(test), 'test examples')
```

```
193 train examples
49 validation examples
61 test examples
```

### 四、使用tf.data创建输入管道

接下来，我们将用`tf.data`包装`dataframe`。这将使我们能够使用`TensorFlow`的`feature columns`作为桥梁，将`Pandas`的`dataframe`中的列映射到用于训练模型的特征。如果我们处理的是一个非常大的CSV文件(大到不能通过内存存储)，我们将使用`tf.data`直接从磁盘读取数据。本教程不讨论这一点。

```python
# 从Pandas的dataframe创建tf.data数据集
def df_to_dataset(df, shuffle=True, batch_size=32):
    df = df.copy()
    labels = df.pop('target')
    ds = tf.data.Dataset.from_tensor_slices((dict(df), labels))
    if shuffle:
        ds = ds.shuffle(buffer_size=len(df))
    ds = ds.batch(batch_size)
    return ds
```

将数据集拆分成小批次（`5`）来演示我们创建的`tf.data`的数据集：

```python
batch_size = 5
train_ds = df_to_dataset(train, batch_size=batch_size)
val_ds = df_to_dataset(val, shuffle=False, batch_size=batch_size)
test_ds = df_to_dataset(test, shuffle=False, batch_size=batch_size)
```

### 五、进一步了解输入管道

现在我们已经创建了输入管道，让我们调用它来查看它返回的数据的格式。我们使用了一个小的批次大小（`5`）来保持输出的可读性。

```python
for feature_batch, label_batch in train_ds.take(1):
    print('Every feature:', list(feature_batch.keys()))
    print('A batch of ages:', feature_batch['age'])
    print('A batch of targets:', label_batch)
```

```
Every feature: ['age', 'sex', 'cp', 'trestbps', 'chol', 'fbs', 'restecg', 'thalach', 'exang', 'oldpeak', 'slope', 'ca', 'thal']
A batch of ages: tf.Tensor([52 40 62 64 37], shape=(5,), dtype=int32)
A batch of targets: tf.Tensor([0 0 0 0 0], shape=(5,), dtype=int32)
```

我们可以看到`tf.data`数据集返回一个字典，`key`为列名(来自`dataframe`)，值映射到`dataframe`中的所有行的列值。

### 六、演示几种类型的特征列（feature column）

`TensorFlow`提供了许多类型的特征列。在本节中，我们将创建几种类型的特征列，并演示它们如何从`dataframe`的列转换为`TensorFlow`的特征列。 

我们将使用第一批次的训练数据进行演示：

```python
example_batch = next(iter(train_ds))[0]
```

下面的工具函数用于创建特征列，并转换一个批次的数据：

```python
def demo(feature_column):
    feature_layer = layers.DenseFeatures(feature_column)
    print(feature_layer(example_batch).numpy())
```

#### 数值列（Numeric columns）

`特征列`的输出会作为模型的输入(使用上面定义的工具函数，我们将清楚地看到来自`dataframe`的每一列是如何转换成特征列的)。`数值列`（numeric column  ）是最简单的特征列类型，它用于表示真实的数值特征。当使用这种特征列时，您的模型将原封不动地从`dataframe`接收列的值。

```python
age = feature_column.numeric_column('age')
demo(age)
```

```
[[52.]
 [40.]
 [62.]
 [64.]
 [37.]]
```

在本教程使用的心脏病数据集中，`dataframe`中的大多数列都是`数值列`。

#### 桶列（Bucketized columns）

通常，您不希望将数值直接输入模型，而是根据数值范围将其值划分为不同的类别。考虑代表一个人年龄的数值数据，我们可以把年龄分成几个阶段，每个阶段称为一个`桶`，形成所谓的`桶列`（`bucketized column`）。

```python
age_buckets = feature_column.bucketized_column(age, boundaries=[18, 25, 30, 35, 40, 45, 50, 55, 60, 65])
demo(age_buckets)
```

```
[[0. 0. 0. 0. 0. 0. 0. 1. 0. 0. 0.]
 [0. 0. 0. 0. 0. 1. 0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 0.]
 [0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 0.]
 [0. 0. 0. 0. 1. 0. 0. 0. 0. 0. 0.]]
```

注意上面输出的是`one-hot`数组，数组中的每一行表示数据中某一行数据代表的某个人的年龄属于哪个年龄范围。

#### 分类列（Categorical columns）

在这个数据集中，`thal`列的值是一个字符串(例如：`fixed`、`normal`或`reversible`)。我们不能将字符串直接提供给模型。相反，我们必须首先将它们转换为数值。分类词汇表列（`categorical vocabulary columns`）提供了一种将字符串表示为一个`one-hot`向量的方法（就像您在上面看到的年龄桶一样）。可以使用`categorical_column_with_vocabulary_list`，将词汇表作为列表传递给该函数。也可以使用`categorical_column_with_vocabulary_file`从文件中加载词汇表。

```python
thal = feature_column.categorical_column_with_vocabulary_list(
        'thal', ['fixed', 'normal', 'reversible'])
 
thal_one_hot = feature_column.indicator_column(thal)
demo(thal_one_hot)
```

```
[[0. 1. 0.]
 [0. 0. 1.]
 [0. 0. 1.]
 [0. 0. 1.]
 [0. 1. 0.]]
```

在更复杂的数据集中，许多列都可能是类似的分类列。在处理这种类型的数据时，使用`feature columns` API是最有效的。

#### 嵌入列（Embedding columns）

假设不是只有几个可能的字符串，而是每个类别有数千个(或更多)值。由于许多原因，随着类别数量的增加，使用`one-hot`编码训练神经网络将变得不再可行。我们可以使用`嵌入列`来克服这个限制。`嵌入列`（`embedding column`）不是将数据表示为有很多维度的`one-hot`向量，而是将该数据表示为一个`低维度`、`密集`的向量，其中每个单元格可以包含任意数字，而不仅仅是`0`或`1`。嵌入的大小(在下面的例子中是`8`)是一个必须调优的参数。 

> **关键点**：当分类列有比较多的可能值时，使用嵌入列是最好的选择。 

我们在这里简单演示一下如何这种方法：

```python
# 注意，传递给嵌入列的是一个分类列
thal_embedding = feature_column.embedding_column(thal, dimension=8)
demo(thal_embedding)
```

```
[[-0.6782604  -0.511632    0.01026161 -0.4221003   0.118221   -0.6251951
  -0.16774575 -0.17677607]
 [ 0.45159644 -0.05390342  0.03806166 -0.6353068  -0.0791701  -0.27596644
   0.49114797 -0.49382535]
 [ 0.45159644 -0.05390342  0.03806166 -0.6353068  -0.0791701  -0.27596644
   0.49114797 -0.49382535]
 [ 0.45159644 -0.05390342  0.03806166 -0.6353068  -0.0791701  -0.27596644
   0.49114797 -0.49382535]
 [-0.6782604  -0.511632    0.01026161 -0.4221003   0.118221   -0.6251951
  -0.16774575 -0.17677607]]
```

#### 散列特征列（Hased feature columns）

另一种表示具有大量值的分类列的方法是使用`categorical_column_with_hash_bucket`，它计算输入的`哈希值`，然后选择一个合适的桶大小（`hash_bucket_size`）对字符串进行编码。在使用这种类型的特征列时，您不需要提供`词汇表`，同时您可以选择让`散列桶`的数量比实际类别的数量小得多，从而节省空间。 

> **关键点**：这种技术的一个重要缺点是可能会有冲突，我们可能会遇到不同的字符串被映射到同一个散列桶的冲突。但实际上，即使存在这种冲突，对于某些数据集，这种方法依然表现得很好。

```python
thal_hased = feature_column.categorical_column_with_hash_bucket(
    'thal', hash_bucket_size=1000)
thal_hased_one_hot = feature_column.indicator_column(thal_hased)
demo(thal_hased_one_hot)
```

```
[[0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]]
```

#### 交叉特征列（Crossed feature columns）

将多个特征组合成一个特征，通常称为`特征交叉`（`feature crosses`）。模型能够为每个组合后的特征学习到单独的权重。在这里，我们将使用`crossed_column`创建一个新的特征，它是`年龄`和`thal`的组合特征。 

> **注意**，`crossed_column`不会构建所有可能组合的完整表（它可能非常大）。相反，它实际上使用了`hashed_column`，因此您可以选择表的大小。

```python
crossed_feature = feature_column.crossed_column([age_buckets, thal], hash_bucket_size=1000)
demo(feature_column.indicator_column(crossed_feature))
```

```
[[0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]
 [0. 0. 0. ... 0. 0. 0.]]
```

### 七、选择合适的特征列

我们已经了解了如何使用几种常见类型的`特征列`。现在我们将用它们来训练一个模型。本教程的目标是向您展示处理特征列所需的完整代码，以及其机制。我们将随意选择一些列来训练我们的模型。 

> **关键点**：如果您的目标是构建一个精确的模型，那么你需要尝试更大的数据集，并仔细考虑哪些特征是最有意义的，以及它们应该被如何表示。

```python
feature_columns = []
 
# numeric columns
for header in ['age', 'trestbps', 'chol', 'thalach', 'oldpeak', 'slope', 'ca']:
    feature_columns.append(feature_column.numeric_column(header))
 
# bucketized columns
age_buckets = feature_column.bucketized_column(age, boundaries=[18, 25, 30, 35, 40, 45, 50, 55, 60, 65])
feature_columns.append(age_buckets)
 
# indicator columns
thal = feature_column.categorical_column_with_vocabulary_list(
        'thal', ['fixed', 'normal', 'reversible'])
thal_one_hot = feature_column.indicator_column(thal)
feature_columns.append(thal_one_hot)
 
# embeddding columns
thal_embeddding = feature_column.embedding_column(thal, dimension=8)
feature_columns.append(thal_embedding)
 
# crossed columns
crossed_feature = feature_column.crossed_column([age_buckets, thal], hash_bucket_size=1000)
crossed_feature = feature_column.indicator_column(crossed_feature)
feature_columns.append(crossed_feature)
```

### 八、创建一个特征层

现在我们已经定义了特征列，接下来，我们将使用`DenseFeatures`创建一个层，该层将被输入到我们的模型。

```python
feature_layer = layers.DenseFeatures(feature_columns)
```

在前面，我们使用了一个小的批次大小（`5`）来演示特征列是如何工作的。这里，我们将创建一个新的输入管道，具有更大批次大小（`32`）。

```python
batch_size = 32
train_ds = df_to_dataset(train, batch_size=batch_size)
val_ds = df_to_dataset(val, shuffle=False, batch_size=batch_size)
test_ds = df_to_dataset(test, shuffle=False, batch_size=batch_size)
```

### 九、创建、编译和训练模型

接下来，我们创建一个模型，并编译和训练它：

```python
model = tf.keras.Sequential([
    feature_layer,
    layers.Dense(128, activation='relu'),
    layers.Dense(128, activation='relu'),
    layers.Dense(1, activation='sigmoid')
])
 
model.compile(optimizer='adam',
              loss='binary_crossentropy',
              metrics=['accuracy'])
 
model.fit(train_ds,
          validation_data=val_ds,
          epochs=5)
```

```
Epoch 1/5
7/7 [==============================] - 0s 71ms/step - loss: 1.0957 - accuracy: 0.6601 - val_loss: 0.6680 - val_accuracy: 0.6735
Epoch 2/5
7/7 [==============================] - 0s 27ms/step - loss: 0.5393 - accuracy: 0.7600 - val_loss: 0.5512 - val_accuracy: 0.6939
Epoch 3/5
7/7 [==============================] - 0s 27ms/step - loss: 0.6062 - accuracy: 0.7403 - val_loss: 0.6553 - val_accuracy: 0.6735
Epoch 4/5
7/7 [==============================] - 0s 27ms/step - loss: 0.5936 - accuracy: 0.7410 - val_loss: 0.7399 - val_accuracy: 0.6735
Epoch 5/5
7/7 [==============================] - 0s 27ms/step - loss: 0.4790 - accuracy: 0.7734 - val_loss: 0.5646 - val_accuracy: 0.7143
<tensorflow.python.keras.callbacks.History at 0x152d5a20>
```

```python
loss, accuracy = model.evaluate(test_ds)
print('Accuracy:', accuracy)
```

```
2/2 [==============================] - 0s 20ms/step - loss: 0.6159 - accuracy: 0.7213
Accuracy: 0.72131145
```

> **关键点**：通常情况下，深度学习在更大更复杂的数据集中才会得到的最佳结果。在处理像本教程的小数据集时，我们建议使用`决策树`或`随机森林`作为基线。本教程的目标不是训练一个精确的模型，而是演示处理结构化数据的机制，从而在将来处理自己的数据集时，可以使用这里的代码作为一个起点。

### 十、下一步

了解`结构化数据`分类的最佳方法是亲自尝试。我们建议您寻找另一个数据集，并使用类似于上面的代码训练一个模型，然后对其进行分类。为了提高精确度，请仔细考虑模型中应该包含哪些特征，以及它们应该被如何表示。 

本教程的完整代码请参考[这里](https://github.com/longyg/machine-learning/blob/master/tensorflow-2.0/4-classify-structured-data.ipynb)。
