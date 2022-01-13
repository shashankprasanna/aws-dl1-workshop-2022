---
title: "3.2 Habana Integrations with TensorFlow"
chapter: false
weight: 5
---


For TensorFlow, Habana integrates the TensorFlow framework with SynapseAI in a plugin using `tf.load_library` and `tf.load_op_library`, calling library modules and custom ops/kernels.

The framework integration includes three main components:

- SynapseAI helpers
- Device
- Graph passes

The TensorFlow framework controls most of the objects required for graph build or graph execution. SynapseAI allows users to create, compile, and launch graph on the device. The Graph passes library optimizes the TensorFlow graph with operations of Pattern Matching, Marking, Segmentation, and Encapsulation (PAMSEN). It is designed to manipulate the TensorFlow graph to fully utilize Gaudi’s HW resources.


{{% notice warning %}}
**To prepare your model, you must load the Habana module libraries first.**
{{% /notice %}}


To load the Habana Module for TensorFlow, you will call `load_habana_module()` located under `library_loader.py`. This function loads the Habana libraries needed to use Gaudi HPU at the TensorFlow level, and these commands can be used:  

```{python}
import tensorflow as tf
from TensorFlow.common.library_loader import load_habana_module
load_habana_module()
```
Once loaded, the Gaudi HPU is registered in TensorFlow and prioritized over CPU. This means that when a given Op is available for both CPU and the Gaudi HPU, the Op is assigned to the Gaudi HPU.  When the model is ported to run on the Gaudi HPU, the software stack decides which ops are placed on the CPU and which are placed on the Gaudi HPU.  The optimization pass automatically places unsupported ops on the CPU.  Ops that do not run on the Gaudi HPU should default to run on the host CPU.    
