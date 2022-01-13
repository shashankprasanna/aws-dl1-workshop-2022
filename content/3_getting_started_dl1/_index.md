---
title: "3. Deep Learning Training with Habana Models"
chapter: false
weight: 3
---

#### In this workshop we will showcase the AWS DL1 instance featuring the Habana(R) Gaudi HPU

This session will introduce users to the new instance and show the step-by-step how to start an instance, run Habana’s
default models, and convert a public model to run on the instance.  We will use Jupyter Notebooks to show these models running on the Instance

We will start with the existing models from the Habana Model References and then use a Public Model to show migration steps

1. Training instances on EC2 DL1 using models on Habana Github [Model-References](https://github.com/HabanaAI/Model-References) Repository
2. Migrating models on to EC2 DL1 and training models using TensorFlow Public Model

For simplicity, the workshop is using the full Habana Deep Learning AMI (DLAMI) image from AWS, which contains the Habana drivers, SynpaseAI (R) Software stack and TensorFlow framework; everything that is needed to run AI workloads.  

At the end of this workshop, the user will be familiar with how to run models on the DL1 instance and be able to do basic model migration.
