---
description: 谷歌团队对于inception net的改进
---

# Rethinking the Inception Architecture for Computer Vision

在论文开始，作者提出了一些通用的调参经验和结构经验。

* 通常而言，网络的表示大小应当从输入到输出递减。因为在网络中加大某一层也并不会带来相比之前更多的信息
* 高维的特征更容易处理，在高维特征上收敛更快。
* 底层的网络中进行通道的汇聚，这样做的损失并不大，作者认为

