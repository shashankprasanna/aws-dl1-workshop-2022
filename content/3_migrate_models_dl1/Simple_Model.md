---
title: "3.1 Notebook: Custom Model Training on CIFAR10"
weight: 1
---

#### Open the following notebook:

`aws-dl1-workshop-2022/notebooks/1_CIFAR10_Jupyter_Notebook`


{{% notice tip %}}
Feel free to follow along with the presenter on livestream
{{% /notice %}}

This Jupyter Notebook example demonstrates how to train a simple neural network on Habana Gaudi<sup>TM</sup> card. The neural network is built with Keras APIs, and trained with CIFAR10 dataset.


We will clone Habana `Model-References` repository 0.15.4 branch to the current directory.


```python
import sys
import tensorflow as tf
tf.compat.v1.disable_eager_execution()
```


```python
from tensorflow.keras import datasets, layers, models

(train_images, train_labels), (test_images, test_labels) = datasets.cifar10.load_data()
train_images, test_images = train_images / 255.0, test_images / 255.0
```


```python
from model_def import get_model
model = get_model(input_shape=(32,32,3))
```

Open `model_def.py` to take a look at our custom model
![](/images/getting_started/cifar1.jpg)

## Code changes to run model on Habana

Clone the Habana Model References repository
```python
!git clone -b 0.15.4 https://github.com/HabanaAI/Model-References.git
```

Add it to system path
```python
sys.path.append('./Model-References')
```

Use `load_habana_module()` function to instruct TensorFlow to run the model training on Habana processor
```python
from TensorFlow.common.library_loader import load_habana_module
load_habana_module()
```
Output:
![](/images/getting_started/cifar2.jpg)

## Train custom model on CIFAR10 dataset


```python
opt = tf.optimizers.Adam(0.001)

model.compile(loss=tf.losses.SparseCategoricalCrossentropy(),
              optimizer=opt,
              metrics=["accuracy"])
```


```python
%%time
model.fit(train_images, train_labels,
          epochs=20,
          batch_size=256,
          validation_data=(test_images, test_labels))
```
