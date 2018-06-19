# Network In Network

###  Network in Network {#network-in-network}

整篇论文相比VGG同样是提出一种基本的网络结构框架，但是区别在于VGG是在传统的卷积网络基础上做的规范化，从而得到更加深的网络提高网络的整体性能。而NIN是提出了两种区别于传统CNN的新思路：mlpconv卷积网络加多层感知器（在实用中相当于1\*1卷积）。使用global average pooling来代替FC。一下就从两个方面分别记录。当然Googlenet更多的是借鉴1\*1卷积的思路。

#### Mlpconv（1\*1卷积） {#mlpconv-1-1-juan-ji}

 NIN文中提出使用mlpconv网络层替代传统的convolution层。mlp层实际上是卷积加传统的mlp（多层感知器），因为convolution是线性的，而mlp是非线性的，后者能够得到更高的抽象，泛化能力更强，与VGG在通过叠加更多的非线性变化来提高网络的表达能力的思路是相通的。在跨通道（cross channel,cross feature map）情况下，mlpconv等价于卷积层+1×1卷积层，所以此时mlpconv层也叫cccp层（cascaded cross channel parametric pooling\)。​

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LCcEWwQDiEvpwesw4Gk%2F-LFLq0njhDAtcJPqOmgI%2F-LFLuDCMT5kai2Fy2-Xj%2Fimage.png?alt=media&token=582be8d7-8657-4078-a14a-979fb5287400)



​​

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LCcEWwQDiEvpwesw4Gk%2F-LFLq0njhDAtcJPqOmgI%2F-LFLv1gQ1XbO3a-SXEfJ%2Fimage.png?alt=media&token=7d0156a5-75f8-4987-8b23-ca00baa8f3c5)

相当于在卷积层输出之后通过1\*1的卷积把各个通道的信息重新组织到一起。这样在1\*1卷积的过程中就可以多叠加非线性性。来达到作者提出的增强繁华能力。

关于1\*1卷积的作用，（ 以下内容摘抄自：[http://www.caffecn.cn/?/question/136](http://www.caffecn.cn/?/question/136)）：

问：发现很多网络使用了1X1卷积核，这能起到什么作用呢？另外我一直觉得，1X1卷积核就是对输入的一个比例缩放，因为1X1卷积核只有一个参数，这个核在输入上滑动，就相当于给输入数据乘以一个系数。不知道我理解的是否正确

答1： 对于单通道的feature map和单个卷积核之间的卷积来说，题主的理解是对的，CNN里的卷积大都是多通道的feature map和多通道的卷积核之间的操作（输入的多通道的feature map和一组卷积核做卷积求和得到一个输出的feature map），如果使用1x1的卷积核，这个操作实现的就是多个feature map的线性组合，可以实现feature map在通道个数上的变化。接在普通的卷积层的后面，配合激活函数，就可以实现network in network的结构了（本内容作者仅授权给CaffeCN社区（caffecn.cn）使用，如需转载请附上内容来源说明。）

答2： 我来说说我的理解，我认为1×1的卷积大概有两个方面的作用吧： 1. 实现跨通道的交互和信息整合 2. 进行卷积核通道数的降维和升维

下面详细解释一下： 1. 这一点孙琳钧童鞋讲的很清楚。1×1的卷积层（可能）引起人们的重视是在NIN的结构中，论文中林敏师兄的想法是利用MLP代替传统的线性卷积核，从而提高网络的表达能力。文中同时利用了跨通道pooling的角度解释，认为文中提出的MLP其实等价于在传统卷积核后面接cccp层，从而实现多个feature map的线性组合，实现跨通道的信息整合。而cccp层是等价于1×1卷积的，因此细看NIN的caffe实现，就是在每个传统卷积层后面接了两个cccp层（其实就是接了两个1×1的卷积层）。 2. 进行降维和升维引起人们重视的（可能）是在GoogLeNet里。对于每一个Inception模块（如下图），原始模块是左图，右图中是加入了1×1卷积进行降维的。虽然左图的卷积核都比较小，但是当输入和输出的通道数很大时，乘起来也会使得卷积核参数变的很大，而右图加入1×1卷积后可以降低输入的通道数，卷积核参数、运算复杂度也就跟着降下来了。以GoogLeNet的3a模块为例，输入的feature map是28×28×192，3a模块中1×1卷积通道为64，3×3卷积通道为128,5×5卷积通道为32，如果是左图结构，那么卷积核参数为1×1×192×64+3×3×192×128+5×5×192×32，而右图对3×3和5×5卷积层前分别加入了通道数为96和16的1×1卷积层，这样卷积核参数就变成了1×1×192×64+（1×1×192×96+3×3×96×128）+（1×1×192×16+5×5×16×32），参数大约减少到原来的三分之一。同时在并行pooling层后面加入1×1卷积层后也可以降低输出的feature map数量，左图pooling后feature map是不变的，再加卷积层得到的feature map，会使输出的feature map扩大到416，如果每个模块都这样，网络的输出会越来越大。而右图在pooling后面加了通道为32的1×1卷积，使得输出的feature map数降到了256。GoogLeNet利用1×1的卷积降维后，得到了更为紧凑的网络结构，虽然总共有22层，但是参数数量却只是8层的AlexNet的十二分之一（当然也有很大一部分原因是去掉了全连接层）。

​

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LCcEWwQDiEvpwesw4Gk%2F-LFLvI-wBYP75oYirTuQ%2F-LFLzGwFnWg6osc4_OpH%2Fimage.png?alt=media&token=9d95dc44-2d2b-4907-b304-640f56c6261a)

 最近大热的MSRA的ResNet同样也利用了1×1卷积，并且是在3×3卷积层的前后都使用了，不仅进行了降维，还进行了升维，使得卷积层的输入和输出的通道数都减小，参数数量进一步减少，如下图的结构。（不然真不敢想象152层的网络要怎么跑起来TAT）​

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LCcEWwQDiEvpwesw4Gk%2F-LFLvI-wBYP75oYirTuQ%2F-LFLzNQHsdJJxfD7JFt9%2Fimage.png?alt=media&token=4a0fd6e4-eb74-4769-9dec-272882187e0e)

### **Global Average Pooling** {#global-average-pooling}

论文里的另一个点是针对传统CNN的FC层提出的改变。传统的CNN中，在底层网络通过卷积层提取特征，例如分类任务中，卷积层之后会将输出送入FC层最后在连接softmax做分类任务。这种网络结构，使得卷积层和传统的神经网络层连接在一起。我们可以把卷积层看做是特征提取器，然后得到的特征再用传统的神经网络进行分类。然而，全连接层因为参数个数太多，往往容易出现过拟合的现象，导致网络的泛化能力不尽人意。于是Hinton采用了Dropout的方法，来提高网络的泛化能力。

论文提出采用全局均值池化的方法，替代传统CNN中的全连接层。与传统的全连接层不同，我们对每个特征图一整张图片进行全局均值池化，这样每张特征图都可以得到一个输出。这样采用均值池化，连参数都省了，可以大大减小网络，避免过拟合，另一方面它有一个特点，每张特征图相当于一个输出特征，然后这个特征就表示了我们输出类的特征。这样如果我们在做1000个分类任务的时候，我们网络在设计的时候，最后一层的特征图个数就要选择1000。但是在文章所做的对比试验中，用池化代替fc并没有带来很大的准确率提升，但是可以大大减少网络参数。

