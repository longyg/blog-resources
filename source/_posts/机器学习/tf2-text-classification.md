---
title: 【TensorFlow 2.0教程】对影视评论进行文本分类
cover: false
top: false
date: 2019-06-04 18:27:48
group: ml
permalink: tf2-text-classification
categories: TensorFlow
tags:
- 机器学习
- TensorFlow
keywords:
- TensorFlow 2 教程
- 文本分类
summary: 本教程将使用 TensorFlow 2.0 训练一个神经网络模型来对影视评论分类
---


本文将对电影评论文本进行分类，分为`正面`影评和`负面`影评，这是一个在机器学习问题中非常重要且常见的`二分类`问题。 

本文演示使用`TensorFlow Hub`和`Keras`进行`转移学习`的基本应用。 

我们将使用`IMDB`数据集，其中包含了来自[互联网电影数据库](https://www.imdb.com/)的50,000篇`电影评论`的文本，它们被分成25000个训练评论文本和25000个测试评论文本。训练集和测试集中的评论类型是比较平衡的，这意味着它们包含相同数量的正面和负面评论。 

我们同样使用`keras`高级API，用于在`TensorFlow`中构建和训练模型。`TensorFlow Hub`是一个用于`转移学习`的库和平台，我们将使用其中已经训练好的文本嵌入模型。 

首先，导入本文将用到的python库：

```python
import numpy as np
import tensorflow as tf
from tensorflow import keras
 
import tensorflow_hub as tfhub
import tensorflow_datasets as tfds
 
print("Version:", tf.__version__)
print("Eager mode:", tf.executing_eagerly())
print("Hub version:", hub.__version__)
print("GPU is", "available" if tf.test.is_gpu_available() else "NOT AVAILABLE")
```

```
Version: 2.0.0-alpha0
Eager mode: True
Hub version: 0.4.0
GPU is NOT AVAILABLE
```

### 下载IMDB数据集

TensorFlow `datasets`库提供了`IMDB`数据集，下面的代码使用`datasets`库下载该数据集：

```python
train_validation_split = tfds.Split.TRAIN.subsplit([6, 4])
 
(train_data, validation_data), test_data = tfds.load(
    name="imdb_reviews", 
    split=(train_validation_split, tfds.Split.TEST),
    as_supervised=True)
```

部分输出如下：

```
Downloading and preparing dataset imdb_reviews (80.23 MiB) to /root/tensorflow_datasets/imdb_reviews/plain_text/0.1.0...
HBox(children=(IntProgress(value=1, bar_style='info', description='Dl Completed...', max=1, style=ProgressStyl…
HBox(children=(IntProgress(value=1, bar_style='info', description='Dl Size...', max=1, style=ProgressStyle(des…
 
 
HBox(children=(IntProgress(value=1, bar_style='info', max=1), HTML(value='')))
HBox(children=(IntProgress(value=0, description='Shuffling...', max=10, style=ProgressStyle(description_width=…
WARNING: Logging before flag parsing goes to stderr.
W0605 04:08:56.784977 140114589456192 deprecation.py:323] From /root/anaconda3/lib/python3.7/site-packages/tensorflow_datasets/core/file_format_adapter.py:247: tf_record_iterator (from tensorflow.python.lib.io.tf_record) is deprecated and will be removed in a future version.
Instructions for updating:
Use eager execution and: 
`tf.data.TFRecordDataset(path)`
HBox(children=(IntProgress(value=1, bar_style='info', description='Reading...', max=1, style=ProgressStyle(des…
HBox(children=(IntProgress(value=0, description='Writing...', max=2500, style=ProgressStyle(description_width=…
...
```

### 探索数据

首先，让我们花点时间来看看数据集的数据格式。每个样本都包含一段`电影评论`文本，以及相应的`标签`。电影评论文本没有经过任何预处理，标签为`0`或`1`的整数值，其中`0`表示`负面`评论，`1`表示`正面`评论。 让我们打印头10个样本看看：

```python
train_examples_batch, train_labels_batch = next(iter(train_data.batch(10)))
train_examples_batch
```

```
<tf.Tensor: id=235, shape=(3,), dtype=string, numpy=
array([b"As a lifelong fan of Dickens, I have invariably been disappointed by adaptations of his novels.<br /><br />Although his works presented an extremely accurate re-telling of human life at every level in Victorian Britain, throughout them all was a pervasive thread of humour that could be both playful or sarcastic as the narrative dictated. In a way, he was a literary caricaturist and cartoonist. He could be serious and hilarious in the same sentence. He pricked pride, lampooned arrogance, celebrated modesty, and empathised with loneliness and poverty. It may be a clich\xc3\xa9, but he was a people's writer.<br /><br />And it is the comedy that is so often missing from his interpretations. At the time of writing, Oliver Twist is being dramatised in serial form on BBC television. All of the misery and cruelty is their, but non of the humour, irony, and savage lampoonery. The result is just a dark, dismal experience: the story penned by a journalist rather than a novelist. It's not really Dickens at all.<br /><br />'Oliver!', on the other hand, is much closer to the mark. The mockery of officialdom is perfectly interpreted, from the blustering beadle to the drunken magistrate. The classic stand-off between the beadle and Mr Brownlow, in which the law is described as 'a ass, a idiot' couldn't have been better done. Harry Secombe is an ideal choice.<br /><br />But the blinding cruelty is also there, the callous indifference of the state, the cold, hunger, poverty and loneliness are all presented just as surely as The Master would have wished.<br /><br />And then there is crime. Ron Moody is a treasure as the sleazy Jewish fence, whilst Oliver Reid has Bill Sykes to perfection.<br /><br />Perhaps not surprisingly, Lionel Bart - himself a Jew from London's east-end - takes a liberty with Fagin by re-interpreting him as a much more benign fellow than was Dicken's original. In the novel, he was utterly ruthless, sending some of his own boys to the gallows in order to protect himself (though he was also caught and hanged). Whereas in the movie, he is presented as something of a wayward father-figure, a sort of charitable thief rather than a corrupter of children, the latter being a long-standing anti-semitic sentiment. Otherwise, very few liberties are taken with Dickens's original. All of the most memorable elements are included. Just enough menace and violence is retained to ensure narrative fidelity whilst at the same time allowing for children' sensibilities. Nancy is still beaten to death, Bullseye narrowly escapes drowning, and Bill Sykes gets a faithfully graphic come-uppance.<br /><br />Every song is excellent, though they do incline towards schmaltz. Mark Lester mimes his wonderfully. Both his and my favourite scene is the one in which the world comes alive to 'who will buy'. It's schmaltzy, but it's Dickens through and through.<br /><br />I could go on. I could commend the wonderful set-pieces, the contrast of the rich and poor. There is top-quality acting from more British regulars than you could shake a stick at.<br /><br />I ought to give it 10 points, but I'm feeling more like Scrooge today. Soak it up with your Christmas dinner. No original has been better realised.",
       b"Oh yeah! Jenna Jameson did it again! Yeah Baby! This movie rocks. It was one of the 1st movies i saw of her. And i have to say i feel in love with her, she was great in this move.<br /><br />Her performance was outstanding and what i liked the most was the scenery and the wardrobe it was amazing you can tell that they put a lot into the movie the girls cloth were amazing.<br /><br />I hope this comment helps and u can buy the movie, the storyline is awesome is very unique and i'm sure u are going to like it. Jenna amazed us once more and no wonder the movie won so many awards. Her make-up and wardrobe is very very sexy and the girls on girls scene is amazing. specially the one where she looks like an angel. It's a must see and i hope u share my interests",
       b"I saw this film on True Movies (which automatically made me sceptical) but actually - it was good. Why? Not because of the amazing plot twists or breathtaking dialogue (of which there is little) but because actually, despite what people say I thought the film was accurate in it's depiction of teenagers dealing with pregnancy.<br /><br />It's NOT Dawson's Creek, they're not graceful, cool witty characters who breeze through sexuality with effortless knowledge. They're kids and they act like kids would. <br /><br />They're blunt, awkward and annoyingly confused about everything. Yes, this could be by accident and they could just be bad actors but I don't think so. Dermot Mulroney gives (when not trying to be cool) a very believable performance and I loved him for it. Patricia Arquette IS whiny and annoying, but she was pregnant and a teenagers? The combination of the two isn't exactly lavender on your pillow. The plot was VERY predictable and but so what? I believed them, his stress and inability to cope - her brave, yet slightly misguided attempts to bring them closer together. I think the characters, acted by anyone else, WOULD indeed have been annoying and unbelievable but they weren't. It reflects the surreality of the situation they're in, that he's sitting in class and she walks on campus with the baby. I felt angry at her for that, I felt angry at him for being such a child and for blaming her. I felt it all.<br /><br />In the end, I loved it and would recommend it.<br /><br />Watch out for the scene where Dermot Mulroney runs from the disastrous counselling session - career performance."],
      dtype=object)>
```

对应的头10个样本的标签：

```python
train_labels_batch
```

```
<tf.Tensor: id=231, shape=(10,), dtype=int64, numpy=array([1, 1, 1, 1, 1, 1, 0, 1, 1, 0])>
```

### 构建模型

`神经网络`模型一般是由多个`层`叠加起来构成的，我们一般需要考虑如下三个主要因素来构建模型:

-   如何`表示`文本？
-   模型中应该使用多少`层`？
-   每层应该有多少个`隐藏单元`（即`神经元`，也称为`节点`）？

在本例中，输入数据由一段文本构成的句子组成，要预测的标签是`0`或`1`。 

表示文本的一种方法是将句子转换成`嵌入向量`。我们可以使用一个预先训练好的文本嵌入模型作为第一层。使用已训练好的文本嵌入模型有以下三个优点：

-   我们不需要考虑文本预处理
-   我们可以从`转移学习`中受益
-   嵌入的大小是固定的，所以处理起来更简单。

在本例中，我们将使用一个来自`TensorFlow Hub`的预先训练好的文本嵌入模型，名为 [google/tf2-preview/gnews-swivel-20dim/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1)。 

`TensorFlow Hub`中还有另外三个其他的训练好的模型，也可以用于本例的测试：

-   [google/tf2-preview/gnews-swivel-20dim-with-oov/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim-with-oov/1) ： 它与[google/tf2-preview/gnews-swivel-20dim/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1)相似，但是有`2.5%`的词汇表转换了`OOV`桶。如果任务的词汇表和模型的词汇表没有完全重叠，使用它将得到更好效果。
-   [google/tf2-preview/nnlm-en-dim50/1](https://tfhub.dev/google/tf2-preview/nnlm-en-dim50/1)： 一个更大的模型，有大约`1M`的词汇量和`50`个维度。
-   [google/tf2-preview/nnlm-en-dim128/1](https://tfhub.dev/google/tf2-preview/nnlm-en-dim128/1)：一个更大的模型，有大约`1M`的词汇量和`128`个维度。

接下来，我们首先创建一个`Keras`层，我们使用`TensorFlow Hub`模型来进行文本嵌入，并对几个输入样本进行测试。注意，无论输入文本的长度如何，文本嵌入的输出形状都是固定的，大小为(`num_examples`, `embedding_dimension`) 。

```python
embedding = "https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1"
hub_layer = hub.KerasLayer(embedding, input_shape=[], 
                           dtype=tf.string, trainable=True)
hub_layer(train_examples_batch[:3])
```

```
<tf.Tensor: id=416, shape=(3, 20), dtype=float32, numpy=
array([[ 3.9819887 , -4.4838037 ,  5.177359  , -2.3643482 , -3.2938678 ,
        -3.5364532 , -2.4786978 ,  2.5525482 ,  6.688532  , -2.3076782 ,
        -1.9807833 ,  1.1315885 , -3.0339816 , -0.7604128 , -5.743445  ,
         3.4242578 ,  4.790099  , -4.03061   , -5.992149  , -1.7297493 ],
       [ 3.4232912 , -4.230874  ,  4.1488533 , -0.29553518, -6.802391  ,
        -2.5163853 , -4.4002395 ,  1.905792  ,  4.7512794 , -0.40538004,
        -4.3401685 ,  1.0361497 ,  0.9744097 ,  0.71507156, -6.2657013 ,
         0.16533905,  4.560262  , -1.3106939 , -3.1121316 , -2.1338716 ],
       [ 3.8508697 , -5.003031  ,  4.8700504 , -0.04324996, -5.893603  ,
        -5.2983093 , -4.004676  ,  4.1236343 ,  6.267754  ,  0.11632943,
        -3.5934832 ,  0.8023905 ,  0.56146765,  0.9192484 , -7.3066816 ,
         2.8202746 ,  6.2000837 , -3.5709393 , -4.564525  , -2.305622  ]],
      dtype=float32)>
```

现在，我们可以使用上面创建的层来构建完整的`神经网络模型`了：

```python
model = tf.keras.Sequential()
model.add(hub_layer)
model.add(tf.keras.layers.Dense(16, activation='relu'))
model.add(tf.keras.layers.Dense(1, activation='sigmoid'))

model.summary()
```

```
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
keras_layer (KerasLayer)     (None, 20)                400020    
_________________________________________________________________
dense (Dense)                (None, 16)                336       
_________________________________________________________________
dense_1 (Dense)              (None, 1)                 17        
=================================================================
Total params: 400,373
Trainable params: 400,373
Non-trainable params: 0
_________________________________________________________________
```

可以看到，我们把几个`层`依次叠加起来，构成了我们的`分类器`模型：

1.  第一层是一个`TensorFlow Hub`层。我们使用了一个预先训练好的被保存起来的模型，通过它将一个句子映射成一个文本`嵌入向量`。这个训练好的模型将句子分割成`标记`（`token`），然后嵌入每个标记，然后组合嵌入形成`嵌入向量`。结果的维度是：(`num_examples`, `embedding_dimension`) 。
2.  第一层输出的固定长度的向量紧接着通过一个有`16`个`隐藏单元`的`全连接`(`Dense`)层。
3.  最后一层是一个单节点`输出层`，使用`sigmoid`激活函数，这个值是一个介于`0`和`1`之间的浮点数，表示一个概率或`置信级别`。

接下来我们需要编译模型。

#### 选择损失函数和优化器

模型需要指定一个`损失函数`和一个用于训练的`优化器`。由于这是一个二元分类问题，并且模型输出一个概率，所以我们将使用`binary_crossentropy`损失函数。 

这不是损失函数的唯一选择，例如，你可以选择`mean_squared_error` （`均方误差`）。但是，一般来说，`binary_crossentropy` 更适合处理概率，它测量概率分布之间的`差距`，在我们的例子中，它表示测量的真实分布和预测之间的`差距`。 

当我们研究`回归问题`(例如，预测房价)时，我们可以使用`均方误差`损失函数。 

现在，为我们的模型配置`优化器`和`损失函数`，同时指定`检测参数`：

```python
model.compile(optimizer='adam',
              loss='binary_crossentropy',
              metrics=['accuracy'])
```

### 训练模型

我们对模型进行`20`次迭代训练，也就是使用训练数据中的所有样本进行`20`次迭代。在训练过程中，对来自验证集的10000个样本进行验证，从而监测模型的`损失`和`准确性`：

```python
history = model.fit(train_data.shuffle(10000).batch(512),
                    epochs=20,
                    validation_data=validation_data.batch(512),
                    verbose=1)
```

部分输出如下：

```
Epoch 1/20
30/30 [==============================] - 7s 245ms/step - loss: 0.7742 - accuracy: 0.4614 - val_loss: 0.0000e+00 - val_accuracy: 0.0000e+00
Epoch 2/20
30/30 [==============================] - 6s 210ms/step - loss: 0.6759 - accuracy: 0.5783 - val_loss: 0.6483 - val_accuracy: 0.6273
Epoch 3/20
30/30 [==============================] - 7s 221ms/step - loss: 0.6084 - accuracy: 0.6863 - val_loss: 0.5836 - val_accuracy: 0.7120
Epoch 4/20
30/30 [==============================] - 7s 219ms/step - loss: 0.5588 - accuracy: 0.7369 - val_loss: 0.5470 - val_accuracy: 0.7414
Epoch 5/20
30/30 [==============================] - 6s 208ms/step - loss: 0.5192 - accuracy: 0.7703 - val_loss: 0.5117 - val_accuracy: 0.7655
...
Epoch 20/20
30/30 [==============================] - 6s 214ms/step - loss: 0.1667 - accuracy: 0.9414 - val_loss: 0.2963 - val_accuracy: 0.8758
```

### 评估模型

使用`evaluate`函数来验证模型对测试集的预测情况。该函数将返回两个值：`损失值`和`准确率`。

```python
results = model.evaluate(test_data.batch(512), verbose=0)
for name, value in zip(model.metrics_names, results):
  print("%s: %.3f" % (name, value))
```

```
25000/25000 [==============================] - 1s 35us/step
[0.32395469411849975, 0.87264]
```

可以看到，使用这种相当简单的方法可以达到约`87%`的准确率。如果采用更高级的方法，模型的准确率应该可以接近`95%`。

### 延伸阅读

要了解处理文本输入的更一般的方法，以及在训练时对准确率和损失值的更详细分析过程，请看[这里](https://tensorflow.google.cn/tutorials/keras/basic_text_classification)。

本文完整代码请参考[这里](https://github.com/longyg/machine-learning/blob/master/tensorflow-2.0/3-classify-text.ipynb)。
