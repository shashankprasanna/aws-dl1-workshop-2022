---
title: "2.1 Download habana Models"
weight: 1
---

#### Training on Habana from the Model-References GitHub repository

Habana has a [Model-References GitHub Page](https://github.com/HabanaAI/Model-References) with a list of reference models that are optimized to run on Gaudi
![](/images/getting_started/habana_gh_header.png)

There are models for TensorFlow and PyTorch covering Computer Vision and NLP usages 

Focus will be on TensorFlow ResNet50 Keras and BERT examples in this workshop.   In general, when using the Full DLAMI, the user will need to to the following to run Habana models: 

1. Clone the GitHub Repository
2. Set the PYTHON ENV Variable
    `export PYTHON=/usr/bin/python3.7` for Ubuntu18.04
3. Download appropriate dataset
4. Select Hyperparameters an options
5. Train and post processing


