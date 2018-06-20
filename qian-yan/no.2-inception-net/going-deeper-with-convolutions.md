---
description: 本文是goolenet的第一个版本
---

# Going deeper with convolutions

在文章开始作者提出了当时卷积神经网络的发展问题：想要提升效果，就会把网络做的越来越big，包括深度与宽度两个层面。但是在网络变大的同时，带来的问题也限制了卷积神经网络的发展，当网络变大的时候参数的上涨是很快速的，由此带来的计算效率的浪费，而且网络变大之后，防止过拟合就变得更加困难，往往需要更大的数据集。但是这并不是合理的发展方向，所以作者提出通过改良结构的办法来提升网络的效果，并且控制网络参数的数量与计算的开销。所以本文的出发点更加的贴近实际应用，而不是单纯的追求在公开数据集比赛上的排名提升。

作者在NIN的启发之下，主要借鉴了1\*1卷积可以减少通道数的特性。设计了一种基本的单元，就是inception。既然可以通过1\*1卷积降低通道数从而大幅减少参数，索性就将很多个不同的基础机构放在inception结构中。最初的naive版本中，在一个inception如下：

![](../../.gitbook/assets/image%20%283%29.png)

在最初版本中，将当时的各种卷积形式，包括证明有助于提高性能的max pooling都包含了进来。但是显而易见的是，在将这些结果concat到一起后，网络的参数会剧增。

这个时候就是之前提到的1\*1卷积来解决这个问题了。改良版的inception如下：

![](../../.gitbook/assets/image%20%284%29.png)

在这一版本中，通过在前一层之后加上1\*1卷积，可以大大减少通道数。这样的话即使concate了很多输入，仍然可以保证较少的参数量。也不会发生网络越深通道越多参数越多的情况。而且就如同NIN中提到的，加入的1\*1卷积部分能够提高模块的非线性表达能力。还有一点，因为在一个inception里包含了多种大小的卷积核，所以每个inception的感受范围就自然地包含了多个尺度。（但是实际上，可以通过VGG论文得到启发，全部采用3\*3大小的卷积核，因为网络会串联多个inception，也可以达到不同感受范围的效果）

之后整个inception net就是将上述提到的inception叠加起来，成为一个完整的深层网络。如下图：

![](../../.gitbook/assets/image%20%287%29.png)

![](../../.gitbook/assets/image%20%282%29.png)

另外在实践环节谷歌团队针对训练细节也提出了一些经验方法。值得记录借鉴一下。

* 学习率表，每8个epoch缩减4%。可能由于ImageNet的数据量太大，在中小数据集或者微调的情况下，可以加大epoch数。
* 提出了一中很激进的图像增强方法。尺度跨越比较大。During testing, we adopted a more aggressive cropping approach than that of Krizhevsky et al. \[9\]. Specifically, we resize the image to 4 scales where the shorter dimension \(height or width\) is 256, 288, 320 and 352 respectively, take the left, center and right square of these resized images \(in the case of portrait images, we take the top, center and bottom squares\). For each square, we then take the 4 corners and the center 224\*224 crop as well as the square resized to 224224, and their mirrored versions. This results in 4\*3\*6\*2 = 144 crops per image.作者也说了在实际使用中减少图像增强的方法也不会带来很大的性能下降。所以在实际训练中，未必需要这么多的图像增强。但是可以借鉴方法。
* 并行的训练若干相同的模型。由于在训练中的送入的图像顺序是随机的，所以训练的结果也会有一定的差别。在计算条件允许的情况下~

