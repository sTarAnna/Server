# 单机器多GPU联合训练指北

本指北参照[Pytorch][1]以及[Tensorflow][2]多GPU联合训练的官方文档

[1]:https://pytorch.org/tutorials/beginner/dist_overview.html
[2]:https://www.tensorflow.org/guide/distributed_training?hl=zh-cn

------

## Contents

- [Pytorch Data Parallel Training](#pytorch-dpt)
- [Tensorflow](#tf)

------

## Pytorch DPT
([↑up to contents](#contents))

### 1. 概述

PyTorch为数据并行训练提供了几种选择。对于从简单到复杂以及从原型到生产逐渐增长的应用程序，共同的发展轨迹将是：
1. 如果数据和模型可以放在一个GPU中，并且不关心训练速度，使用单设备训练。
2. 如果服务器上有多个GPU，使用单机多GPU [DataParallel][3]，并且希望以最少的代码更改来加快培训速度。
3. 如果想进一步加快培训速度并愿意编写更多代码来进行设置，请使用单机多GPU [DistributedDataParallel][4]。
4. 如果应用程序需要跨计算机边界扩展，使用多计算机[DistributedDataParallel][4]和相应[脚本][5]。
5. 如果预计会出现错误（例如，OOM），或者在训练期间资源可以动态加入和离开，使用[torchelastic][6]启动分布式培训。

[3]:https://pytorch.org/docs/master/generated/torch.nn.DataParallel.html
[4]:https://pytorch.org/docs/master/generated/torch.nn.parallel.DistributedDataParallel.html
[5]:https://github.com/pytorch/examples/blob/master/distributed/ddp/README.md
[6]:https://pytorch.org/elastic/0.2.1/index.html

虽然采用第二种DataParallel API编程的方式改动较少，但其实现为单进程多线程的方式，由于Python解释器中的全局锁（GIL）会影响到训练的并行速度，本文推荐并主要介绍采用第三种方式DistributedDataParallel API方式的代码编写。

### 2. GPU并行训练原理

从深度学习目前的基础之一：mini-batch gradient descent training（小批量梯度下降学习）的原理出发，在单卡上训练的时候模型计算整个batch的平均loss并进行梯度下降。在多张卡上进行训练时所做过程与单卡类似，只是每张卡处理的数据不同。具体来说，进行N个GPU并行训练时，要在每个GPU上加载相同参数的模型，并将一个batch拆成N部分分发给每个GPU上的模型，每个模型进行一次正向计算得到本地梯度，接下来，把N块显卡的显存上的本地梯度相加，得到当前的batch的随机梯度。之后，每块GPU都使用这个随机梯度更新该GPU显存所维护的模型参数（独立更新完后模型参数仍然保持一致）。
####Tips
GPU并行训练的还有一种方式是将一个庞大模型（单个GPU放不下）放在两个GPU上进行训练。

### 3. Pytorch DistributedDataParallel 详解

在一次
#### 






