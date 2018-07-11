---
description: Res-net理论基础
---

# Deep Residual Learning for Image Recognition

Res-net在ImageNet的classification、detection、localization以及COCO的detection和segmentation上均获得了第一名的成绩。

#### 问题提出

论文主要是针对现有模型的深度加深后出现的退化问题作出的改善。如下图，当模型的深度加深后反倒出现了准确率下降的问题。

![](../.gitbook/assets/image%20%289%29.png)

通过观察实验结果得出其实在模型的层数加多之后，整个网络的梯度出现了消失/弥散的情况，导致网络的优化困难。（按照之前的论文来看，随着网络的加深，整个网络的表达能力应该更强，所以解决了梯度消失的优化问题就能够通过加深网络来得到更好的网络性能）

#### Residual结构

![](../.gitbook/assets/image%20%2814%29.png)

F\(x\) = H\(x\)-x（H\(x\)代表原来的层的数学抽象）作者受到图像处理中残差向量编码VLAD的启发， 通过一个reformulation，将一个问题分解成多个尺度直接的残差问题，能够很好的起到优化训练的效果。所以认为采用F\(x\)+x代替H\(x\)会比较容易优化，并且两种的效果相同。

这个Residual block通过shortcut connection实现，通过shortcut将这个block的输入和输出进行一个element-wise的加叠，这个简单的加法并不会给网络增加额外的参数和计算量，同时却可以大大增加模型的训练速度、提高训练效果，并且当模型的层数加深时，这个简单的结构能够很好的解决退化问题。

