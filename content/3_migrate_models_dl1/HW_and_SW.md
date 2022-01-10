---
title: "3.1 Habana Hardware and Software"
chapter: false
weight: 1
---


### Hardware
Gaudi is designed from the ground up for accelerating DL training workloads. Its heterogeneous architecture comprises a cluster of fully programmable Tensor Processing Cores (TPC) along with its associated development tools and libraries, and a configurable Matrix Math engine.

The TPC core is a VLIW SIMD processor with instruction set and hardware tailored to serve training workloads efficiently. It is programmable, providing the user with maximum flexibility to innovate, coupled with many workload-oriented features, such as:

- GEMM operation acceleration
- Tensor addressing
- Latency hiding capabilities
- Random number generation
- Advanced implementation of special functions

The TPC core natively supports the following data types: FP32, BF16, INT32, INT16, INT8, UINT32, UINT16 and UINT8.

The Gaudi memory architecture includes on-die SRAM and local memories in each TPC. In addition, the chip package integrates four HBM devices, providing 32 GB of capacity and 1 TB/s bandwidth. The PCIe interface provides a host interface and supports both generation 3.0 and 4.0 modes.

Gaudi is the first DL training processor that has integrated RDMA over Converged Ethernet (RoCE v2) engines on-chip. With bi-directional throughput of up to 2 TB/s, these engines play a critical role in the inter-processor communication needed during the training process. This native integration of RoCE allows customers to use the same scaling technology, both inside the server and rack (scale-up), as well as to scale across racks (scale-out). These can be connected directly between Gaudi processors, or through any number of standard Ethernet switches.

![](/images/migrate/gaudi_HW_arch.png)

### Software
Designed to facilitate high-performance DL training on Habana’s Gaudi accelerators, SynapseAI Software Suite enables efficient mapping of neural network topologies onto Gaudi hardware. The software suite includes Habana’s graph compiler and runtime, TPC kernel library, firmware and drivers, and developer tools such as the TPC SDK for custom kernel development and SynapseAI Profiler. SynapseAI is integrated with popular frameworks, TensorFlow and PyTorch, and performance-optimized for Gaudi

![](/images/migrate/gaudi_SW_arch.png)

#### Graph Compiler and Runtime
The SynapseAI graph compiler generates optimized binary code that implements the given model topology on Gaudi. It performs operator fusion, data layout management, parallelization, pipelining and memory management, as well as graph-level optimizations. The graph compiler uses the rich TPC kernel library which contains a wide variety of operations (for example, elementwise, non-linear, non-GEMM operators). Kernels for training have two implementations, forward and backward.

Given the heterogeneous nature of Gaudi hardware (Matrix Math engine, TPC and DMA), the SynapseAI graph compiler enables effective utilization through parallel and pipelined execution of framework graphs. SynapseAI uses stream architecture to manage concurrent execution of asynchronous tasks. It includes a multi-stream execution environment supporting Gaudi’s unique combination of compute and networking as well as exposing a multi-stream architecture to the framework. Streams of different types — compute, networking and DMA — are synchronized with one another at high performance and with low run-time overheads.

#### DL Framework Integration
Popular DL frameworks such as TensorFlow and PyTorch are integrated with SynapseAI and optimized for Gaudi. SynapseAI does this under the hood, so customers still enjoy the same abstraction in TensorFlow and PyTorch that they are accustomed to today. The SynapseAI TensorFlow/PyTorch bridge identifies the subset of the framework’s computation graph that can be accelerated by Gaudi. These subgraphs are executed optimally on Gaudi. For performance optimization, the compilation recipe is cached for future use. Operators that are not supported by Gaudi are executed on the CPU.

#### Habana Communication Libraries
The Habana Communication Library (HCL) enables efficient scale-up communication between Gaudi processors within a single node and scale-out across nodes for distributed training, leveraging Gaudi’s high performance RDMA communication capabilities. It has an MPI look-and-feel and supports point-to- point operations (for example, Write, Send) and collective operations (for example, AllReduce, AlltoAll) that are performance optimized for Gaudi. See Habana Communication Library (HCL) API Reference for further details.

The SynapseAI suite main interface is the Habana Collective Communications Library (HCCL) which is Habana’s implementation of standard collective communication routines with NCCL-compatible API. HCL uses Gaudi integrated NICs for both scale-up and scale-out. HCCL allows users to enable Gaudi integrated NIC for scale-up and host NIC for scale-out. 

#### TPC Programming
The SynapseAI TPC SDK includes an LLVM-based TPC-C compiler, a simulator and debugger. These tools facilitate the development of custom TPC kernels. This SDK is used by Habana to build the high-performance kernels we provide to users. You can thereby develop customized deep learning models and algorithms on Gaudi to innovate and optimize to your unique requirements.


