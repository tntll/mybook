---
description: 谷歌团队对于inception net的改进
---

# Rethinking the Inception Architecture for Computer Vision

在论文开始，作者提出了一些通用的**调参经验和结构经验**。

* 通常而言，网络的表示大小应当从输入到输出递减。因为在网络中加大某一层也并不会带来相比之前更多的信息
* 高维的特征更容易处理，在高维特征上收敛更快。
* 底层的网络中进行通道的汇聚，这样做的损失并不大，作者认为底层的神经元之间的相关性较大，存在一定的冗余
* 应当平衡网络的深度和宽度，平衡的网络质量更高，并且在分布式计算中有合理有效的 computational budget

#### Smaller convolutions

在之前的inception net中每个inception含有集中不同大小的卷积核：3\*3，5\*5，7\*7。类比VGG的网络结构。其实就是通过将不同大小的卷积核都采用一系列的3\*3卷积核的叠加来替代。这样既可以达到相同的感受野范围，又可以减少参数。

![](../../.gitbook/assets/image%20%285%29.png)

这样做之后每层的3\*3卷积之后都加上了relu函数。作者通过实验证明，减少参数并没有带来性能的下降，还有加入relu函数的效果总比不加好一些。（如果参考VGG论文，因为叠加了更多的非线性函数，反而能够给网络带来更大的非线性表达能力，提高网络的性能）。

#### 更小的卷积核

在使用3\*3卷积核之后，作者提出了是否可以采用更加小的卷积核，2\*2，或者1\*n，n\*1这样的结构。例如通过1\*3和3\*1的叠加来代替3\*3卷积核。采用这种方式可以将参数减少33%。如下图

![](../../.gitbook/assets/image%20%281%29.png)

但是这种结构在实际操作中，在网络的浅层采用这种方式并没有带来很好的效果。在网络的中层，即feature map大小适中的时候采用这种结构，效果会比较好。这样inception的结构就变成了如下图：

![](../../.gitbook/assets/image%20%289%29.png)

#### Utility of Auxiliary Classifiers

在之前的inception net中提出了增加额外旁支分类器来提高网络的训练结果。使用了多余的在底层的分类器，直觉上可以认为这样做可以使底层能够在梯度下降中学的比较充分，但在实践中发现两条：

* 多余的分类器在训练开始的时候并不能起到作用，在训练快结束的时候，使用它可以有所提升
* 最底层的那个多余的分类器去掉以后也不会有损失。
* 以为多余的分类器起到的是梯度传播下去的重要作用，但通过实验认为实际上起到的是regularizer的作用，因为在多余的分类器前添加dropout或者batch normalization后效果更佳。（引用[https://blog.csdn.net/stdcoutzyx/article/details/51052847](https://blog.csdn.net/stdcoutzyx/article/details/51052847)）

#### Efficient Grid Size Reduction

为了缩小feature\_map的大小，一般有两种方法。

![](../../.gitbook/assets/image%20%2817%29.png)

左图表示传统的方法是直接采用pooling操作，但正如之前所述，直接采用pooling会导致特征的表征遇到瓶颈。右图所示的方法虽然可以减少特征减少问题，但是带来了很大的额外计算量。 为了同时达到不违反规则且降低计算量的作用，将网络改为下图：

![](../../.gitbook/assets/image%20%2811%29.png)

采用两路并行的计算来降低计算量并且避免降低特征表达能力。

最后整个inception模型的V2版本结构如下

![](../../.gitbook/assets/image%20%287%29.png)

 图中的Figure 4是指没有进化的Inception，Figure 5是指smaller conv版的Inception，Figure 6是指Asymmetric版的Inception

#### Model Regularization via Label Smoothing

最后作者还提出了一个关于目标函数的改进方法。之前的目标函数在单类的情况下，如果某一类概率接近1，其他的概率接近0，那么会导致交叉熵取log后变得很大很大。从而导致两个问题：

* 过拟合
* 导致样本属于某个类别的概率非常的大，模型太过于自信自己的判断。

所以，使用了一种平滑方法，可以使得类别概率之间的差别没有那么大：

![](../../.gitbook/assets/image%20%288%29.png)

相当于给每一项中加入了一个均匀分布的子项，来缩减单纯的1与0带来的问题。使得目标函数可以相对平滑。

![](../../.gitbook/assets/image%20%2813%29.png)

这项改动可以提升0.2%的效果。

