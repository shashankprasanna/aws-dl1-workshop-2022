---
title: "3.4 Notebook: Migrating TensorFlow EfficientNet to Habana Gaudi"
weight: 30
---

#### Open the following notebook:

`aws-dl1-workshop-2022/notebooks/2_EfficientNet_Jupyter_Notebook`

In this session, we will learn how to migrate EfficientNet in public
TensorFlow [model garden](https://github.com/tensorflow/models/tree/master/official/vision/image_classification) to
Habana Gaudi device with very limited code changes. We will first clone the model garden repository to local disk, and
enable the model training on CPU by modifying an existing configuration file. Then we will add code to the training
script to load Habana software modules and enable it on HPU.

First of all, let's check the current directory to prepare for cloning TensorFlow models repository.

    %pwd

Then, we will clone TensorFlow [models](https://github.com/tensorflow/models.git)  repository to the current directory.

```python
!git clone https://github.com/tensorflow/models.git
```
The following output indicates the repository was successfully cloned:

    Cloning into 'models'...
    remote: Enumerating objects: 66323, done.
    remote: Counting objects: 100% (32/32), done.
    remote: Compressing objects: 100% (26/26), done.
    remote: Total 66323 (delta 6), reused 15 (delta 4), pack-reused 66291
    Receiving objects: 100% (66323/66323), 575.88 MiB | 66.89 MiB/s, done.
    Resolving deltas: 100% (46471/46471), done.


We need to download Habana software packages as the dependency to enable TensorFlow EfficientNet on HPU.
If you haven't cloned Habana [Model-References](https://github.com/HabanaAI/Model-References.git)
repository branch 0.15.4 to the current directory, do it with the following command:


```python
!git clone -b 0.15.4 https://github.com/HabanaAI/Model-References.git
```

The following output indicates the repository was successfully cloned:

    Cloning into 'Model-References'...
    remote: Enumerating objects: 5011, done.
    remote: Counting objects: 100% (2786/2786), done.
    remote: Compressing objects: 100% (2035/2035), done.
    remote: Total 5011 (delta 1148), reused 2149 (delta 696), pack-reused 2225
    Receiving objects: 100% (5011/5011), 64.05 MiB | 50.72 MiB/s, done.
    Resolving deltas: 100% (1944/1944), done.


Check if the current PYTHONPATH contains TensorFlow `models` location and Habana `Model-References` location.
If not, add them to PYTHONPATH:


```python
%env PYTHONPATH
```

If PYTHONPATH doesn't exist or doesn't include TensorFlow `models` repository and Habana `Model-References` repository locations,
then add them. The following command assumes the repositories were cloned to `/home/ubuntu/` directory. Modify it accordingly if they are located in a difference folder.


```python
%set_env PYTHONPATH=/home/ubuntu/aws-dl1-workshop-2022/notebooks/3_EfficientNet_Jupyter_Notebook/Model-References:/home/ubuntu/aws-dl1-workshop-2022/notebooks/3_EfficientNet_Jupyter_Notebook/models
```

Verify if the repository locations were added to the PYTHONPATH with the command above.

```python
%env PYTHONPATH
```

We will be using TensorFlow Keras EfficientNet at https://github.com/tensorflow/models/tree/master/official/vision/image_classification
as the example to demonstrate how to enable a public model on Habana Gaudi device.

EfficientNet is a convolutional neural network architecture and scaling method that uniformly scales all
dimensions of depth/width/resolution using a compound coefficient. The model was first introduced by
Tan et al. in [EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks](https://arxiv.org/abs/1905.11946).  
In this session, we are going to use EfficientNet baseline model EfficientNet-B0 as the training workload.

First of all, let's enable the training with synthetic data on CPU and check its performance.

Let's go to `EfficientNet` model directory and check its contents:

```python
%cd models/official/vision/image_classification
%ls
```

The folder with `EfficientNet` model should contain files as below:

    README.md                        efficientnet/
    __init__.py                      learning_rate.py
    __pycache__/                     learning_rate_test.py
    augment.py                       mnist_main.py
    augment_test.py                  mnist_test.py
    callbacks.py                     optimizer_factory.py
    classifier_trainer.py            optimizer_factory_test.py
    classifier_trainer_test.py       preprocessing.py
    classifier_trainer_util_test.py  resnet/
    configs/                         test_utils.py
    dataset_factory.py


In TensorFlow `models` repository, there are only EfficientNet configuration files for GPU and TPU under `configs` directory. We need to modify an existing GPU configuration file to enable it on CPU.

Open [configs/examples/efficientnet/imagenet/efficientnet-b0-gpu.yaml](../edit/models/official/vision/image_classification/configs/examples/efficientnet/imagenet/efficientnet-b0-gpu.yaml)
by clicking the link or manually navigate to `configs/examples/efficientnet/imagenet/efficientnet-b0-gpu.yaml`, and modify the contents with the suggestions as below:

   * Line 6:  change distribution_strategy to `off`
   * Line 7:  change num_gpus to `0`
   * Line 11: change builder to `synthetic`
   * Line 23: change builder to `synthetic`
   * Line 50: insert `steps: 1000`
   * Line 51: change epochs to `1`
   * Line 53: insert `skip_eval: True`

Save the file.


The modified configuration file looks like the following:
![efficientnet_config](/images/migrate/enet_config.png)

After we modify the EfficientNet configuration file above, we can run the following command to launch the training on CPU for 1000 iterations. We will skip evaluations in order to focus on training. Check the throughput for performance in the output log.


```python
!$PYTHON classifier_trainer.py --mode=train_and_eval --model_type=efficientnet --dataset=imagenet --model_dir=$HOME/log_cpu --data_dir=$HOME --config_file=configs/examples/efficientnet/imagenet/efficientnet-b0-gpu.yaml
```

Training on CPU output results are:

    2021-11-05 21:20:01.119219: I tensorflow/compiler/mlir/mlir_graph_optimization_pass.cc:176] None of the MLIR Optimization Passes are enabled (registered 2)
    2021-11-05 21:20:01.273872: I tensorflow/core/platform/profile_utils/cpu_utils.cc:114] CPU Frequency: 3000005000 Hz
    I1105 21:21:53.864213 140672312903488 keras_utils.py:148] TimeHistory: 111.52 seconds, 28.69 examples/second between steps 0 and 100
    I1105 21:23:11.292150 140672312903488 keras_utils.py:148] TimeHistory: 77.43 seconds, 41.33 examples/second between steps 100 and 200
    I1105 21:24:27.747459 140672312903488 keras_utils.py:148] TimeHistory: 76.45 seconds, 41.86 examples/second between steps 200 and 300
    I1105 21:25:43.755411 140672312903488 keras_utils.py:148] TimeHistory: 76.01 seconds, 42.10 examples/second between steps 300 and 400
    I1105 21:26:58.760426 140672312903488 keras_utils.py:148] TimeHistory: 75.00 seconds, 42.66 examples/second between steps 400 and 500
    I1105 21:28:13.050693 140672312903488 keras_utils.py:148] TimeHistory: 74.29 seconds, 43.08 examples/second between steps 500 and 600
    I1105 21:29:28.185031 140672312903488 keras_utils.py:148] TimeHistory: 75.13 seconds, 42.59 examples/second between steps 600 and 700
    I1105 21:30:42.560744 140672312903488 keras_utils.py:148] TimeHistory: 74.37 seconds, 43.03 examples/second between steps 700 and 800
    I1105 21:31:57.281210 140672312903488 keras_utils.py:148] TimeHistory: 74.72 seconds, 42.83 examples/second between steps 800 and 900
    I1105 21:33:11.633263 140672312903488 keras_utils.py:148] TimeHistory: 74.35 seconds, 43.04 examples/second between steps 900 and 1000
    1000/1000 - 789s - loss: 1.8846 - accuracy: 0.9445 - top_5_accuracy: 0.9477

    Epoch 00001: saving model to /home/ubuntu/log_cpu/model.ckpt-0001
    I1105 21:33:11.898042 140672312903488 classifier_trainer.py:445] Run stats:
    {'loss': 1.8845537900924683, 'training_accuracy_top_1': 0.9444687366485596, 'step_timestamp_log': ['BatchTimestamp<batch_index: 0, timestamp: 1636147202.340956>', 'BatchTimestamp<batch_index: 100, timestamp: 1636147313.8640285>', 'BatchTimestamp<batch_index: 200, timestamp: 1636147391.2921143>', 'BatchTimestamp<batch_index: 300, timestamp: 1636147467.747421>', 'BatchTimestamp<batch_index: 400, timestamp: 1636147543.7553766>', 'BatchTimestamp<batch_index: 500, timestamp: 1636147618.7602837>', 'BatchTimestamp<batch_index: 600, timestamp: 1636147693.050658>', 'BatchTimestamp<batch_index: 700, timestamp: 1636147768.1849937>', 'BatchTimestamp<batch_index: 800, timestamp: 1636147842.5607076>', 'BatchTimestamp<batch_index: 900, timestamp: 1636147917.2811751>', 'BatchTimestamp<batch_index: 1000, timestamp: 1636147991.6332252>'], 'train_finish_time': 1636147991.831136, 'avg_exp_per_second': 40.53250048399817}


From the output log above, we can see that the throughput for EfficientNet-B0 training on CPU with synthetic data is around `42 examples/sec`.

Now, let's modify the traning script and enable the model on Habana Gaudi device.

Open [classifier_trainer.py](../edit/models/official/vision/image_classification/classifier_trainer.py) by clicking the link and edit it according to the instructions as below.

   * Line 443:  Insert the following 3 lines of code:
   ```
   from TensorFlow.common.library_loader import load_habana_module
log_info_devices = load_habana_module()
logging.info('Devices:\n%s', log_info_devices)
   ```

The modified training script looks like the following:

![enet_script](/images/migrate/enet_script.png)

Save the file.

**Note**: if you use [Model-References](https://github.com/HabanaAI/Model-References.git) repository version 1.1.0 or later,
you need to import `load_habana_module` by using:

    from habana_frameworks.tensorflow import load_habana_module

The 3 lines code above will load Habana software modules so that Habana Gaudi device will be aquired in the beginning of workload running,
and EfficientNet training will be deployed and performed on Habana Gaudi device. This is all you need to do to enable EfficientNet on HPU.

Now, run the following command to launch the training on HPU and check the performance in the output.


```python
!$PYTHON classifier_trainer.py --mode=train_and_eval --model_type=efficientnet --dataset=imagenet --model_dir=$HOME/log_hpu --data_dir=$HOME --config_file=configs/examples/efficientnet/imagenet/efficientnet-b0-gpu.yaml
```

Output results for training on HPU are:


    2021-11-08 19:19:04.089074: I /home/jenkins/workspace/cdsoftwarebuilder/create-tensorflow-module---bpt-d/tensorflow-training/habana_device/kernels/hpu_write_summary.h:148] Plugin graph_keras_modelis not supported yet. skipping..
    2021-11-08 19:19:04.512995: I tensorflow/compiler/mlir/mlir_graph_optimization_pass.cc:176] None of the MLIR Optimization Passes are enabled (registered 2)
    2021-11-08 19:19:04.711418: I tensorflow/core/platform/profile_utils/cpu_utils.cc:114] CPU Frequency: 3000005000 Hz
    I1108 19:20:33.232040 140100565358400 keras_utils.py:148] TimeHistory: 87.55 seconds, 36.55 examples/second between steps 0 and 100
    I1108 19:20:42.155675 140100565358400 keras_utils.py:148] TimeHistory: 8.92 seconds, 358.78 examples/second between steps 100 and 200
    I1108 19:20:51.057700 140100565358400 keras_utils.py:148] TimeHistory: 8.90 seconds, 359.60 examples/second between steps 200 and 300
    I1108 19:20:59.985192 140100565358400 keras_utils.py:148] TimeHistory: 8.92 seconds, 358.59 examples/second between steps 300 and 400
    I1108 19:21:08.887536 140100565358400 keras_utils.py:148] TimeHistory: 8.90 seconds, 359.58 examples/second between steps 400 and 500
    I1108 19:21:17.781768 140100565358400 keras_utils.py:148] TimeHistory: 8.89 seconds, 359.92 examples/second between steps 500 and 600
    I1108 19:21:26.675821 140100565358400 keras_utils.py:148] TimeHistory: 8.89 seconds, 359.92 examples/second between steps 600 and 700
    I1108 19:21:35.611715 140100565358400 keras_utils.py:148] TimeHistory: 8.93 seconds, 358.23 examples/second between steps 700 and 800
    I1108 19:21:44.458321 140100565358400 keras_utils.py:148] TimeHistory: 8.84 seconds, 361.84 examples/second between steps 800 and 900
    I1108 19:21:53.363223 140100565358400 keras_utils.py:148] TimeHistory: 8.90 seconds, 359.48 examples/second between steps 900 and 1000
    1000/1000 - 168s - loss: 1.9151 - accuracy: 0.9455 - top_5_accuracy: 0.9490

    Epoch 00001: saving model to /home/ubuntu/log_hpu/model.ckpt-0001
    I1108 19:21:53.773185 140100565358400 classifier_trainer.py:448] Run stats:
    {'loss': 1.9150632619857788, 'training_accuracy_top_1': 0.9454999566078186, 'step_timestamp_log': ['BatchTimestamp<batch_index: 0, timestamp: 1636399145.6796935>', 'BatchTimestamp<batch_index: 100, timestamp: 1636399233.2318594>', 'BatchTimestamp<batch_index: 200, timestamp: 1636399242.155496>', 'BatchTimestamp<batch_index: 300, timestamp: 1636399251.0575402>', 'BatchTimestamp<batch_index: 400, timestamp: 1636399259.9850357>', 'BatchTimestamp<batch_index: 500, timestamp: 1636399268.8873763>', 'BatchTimestamp<batch_index: 600, timestamp: 1636399277.7816088>', 'BatchTimestamp<batch_index: 700, timestamp: 1636399286.67566>', 'BatchTimestamp<batch_index: 800, timestamp: 1636399295.6115632>', 'BatchTimestamp<batch_index: 900, timestamp: 1636399304.4581752>', 'BatchTimestamp<batch_index: 1000, timestamp: 1636399313.3630521>'], 'train_finish_time': 1636399313.7082312, 'avg_exp_per_second': 190.44414397784678}


From the output log above, we can see that the throughput for EfficientNet-B0 training on Habana Gaudi with synthetic data is around `360 examples/sec`.
