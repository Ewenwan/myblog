---
title: 神经网络的压缩优化方法总结

date: 2017/8/5 12:04:12

categories:
- 深度学习
tags:
- deeplearning
- 网络优化
- 神经网络
---

这里回顾一下几个经典模型，我们主要看看深度和caffe模型大小，[神经网络模型演化](https://dragonfive.github.io/2017-07-05/deep_learning_model/)。并总结一些模型压缩优化的方法。

![各种CNN模型][1]

<!--more-->

模型大小(参数量)和模型的深浅并非是正相关。



# 一些经典的模型-修改网络架构

## fully connect to local connect 全连接到卷积神经网络  1x1卷积
Alexnet[1]是一个8层的卷积神经网络，有约**60M个参数**，如果采用**32bit float存下来有200M**。值得一提的是，AlexNet中仍然有3个全连接层，其参数量占比参数总量超过了90%。

下面举一个例子，假如输入为28×28×192，输出feature map通道数为128。那么，直接接3×3卷积，参数量为3×3×192×128=221184。

如果先用**1×1卷积进行降维到96个通道**，然后再用3×3升维到128，则参数量为：1×1×192×96+3×3×96×128=129024，参数量减少一半。虽然参数量减少不是很明显，但是如果1×1输出维度降低到48呢？则参数量又减少一半。对于上千层的大网络来说，效果还是很明显了。

移动端对模型大小很敏感。下载一个100M的app与50M的app，首先用户心理接受程度就不一样。

原则上降低通道数是会降低性能的，这里为什么却可以降维呢？我们可以从很多embedding技术，比如PCA等中得到思考，**降低一定的维度可以去除冗余数据**，损失的精度其实很多情况下都不会对我们解决问题有很大影响。

1×1卷积，在 GoogLeNet Inception v1以及后续版本，ResNet中都大量得到应用，**有减少模型参数**的作用。

## 卷积拆分 

(1) VGG

VGG可以认为是AlexNet的增强版，**两倍的深度，两倍的参数量**。不过，也提出了一个模型压缩的trick，后来也被广泛借鉴。

那就是，**对于5×5的卷积，使用两个3×3的卷积串联**，可以得到同样的感受野，但参数量却有所降低，为3×3×2/(5×5)=0.72，同样的道理3个3×3卷积代替一个7×7，则参数压缩比3×3×3/(7×7)=0.55，降低一倍的参数量，也是很可观的。

(2) GoogLeNet

GoogleLet Inception v2就借鉴了VGG上面的思想。而到了Inception V3网络，则更进一步，将大卷积分解(Factorization)为小卷积。

比如**7×7的卷积，拆分成1×7和7×1**的卷积后。参数量压缩比为1×7×2/(7×7)=0.29，比上面拆分成3个3×3的卷积，更加节省参数了。

问题是这种**非对称的拆分**，居然比对称地拆分成几个小卷积核改进效果更明显，**增加了特征多样性**。

后来的Resnet就不说了，也是上面这些trick。到现在，基本上网络中都是3×3卷积和1×1卷积，5×5很少见，7×7几乎不可见。

(3) SqueezeNet

squeezenet将上面1×1降维的思想进一步拓展。通过减少3×3的filter数量，将其一部分替换为1×1来实现压缩。

具体的一个子结构如下：一个squeeze模块加上一个expand模块，使squeeze中的通道数量，少于expand通道数量就行。

![squeezenet网络的expand模块][2]

假如输入为M维，如果直接接3×3卷积，输出为7个通道，则参数量：M×3×3×7。

如果按上图的做法，则参数量为M×1×1×3+3×4×1×1+3×4×3×3，压缩比为：(40+M)/21M，当M比较大时，约0.05。

文章最终**将AlexNet压缩到原来1/50**，而性能几乎不变。

---

SqueezeNet的核心指导思想是——在保证精度的同时使用最少的参数。而这也是所有模型压缩方法的一个终极目标。

基于这个思想，SqueezeNet提出了3点网络结构设计策略：

策略 1. 将3x3卷积核替换为1x1卷积核。

这一策略很好理解，因为1个1x1卷积核的参数是3x3卷积核参数的1/9，这一改动理论上可以将模型尺寸压缩9倍。

策略 2. 减小输入到3x3卷积核的输入通道数。

我们知道，对于一个采用3x3卷积核的卷积层，该层所有卷积参数的数量（不考虑偏置）为：
![enter description here][3]

N是卷积核的数量，也即输出通道数，C是输入通道数。因此，为了保证减小网络参数，**不仅仅需要减少3x3卷积核的数量，还需减少输入到3x3卷积核的输入通道数量**，即式中C的数量。

策略 3.尽可能的将降采样放在网络后面的层中。

分辨率越大的特征图（延迟降采样）可以带来更高的分类精度，而这一观点从直觉上也可以很好理解，因为分辨率越大的输入能够提供的信息就越多。

上述三个策略中，前两个策略都是针对如何降低参数数量而设计的，最后一个旨在最大化网络精度。


**squeeze层**借鉴了inception的思想，利用1x1卷积核来降低输入到expand层中3x3卷积核的输入通道数。

定义squeeze层中1x1卷积核的数量是$s_{1*1}$，类似的，expand层中1x1卷积核的数量是$e_{1*1}$， 3x3卷积核的数量是$e_3*3$。令$s_{1*1} < e_{1*1}+ e_{3*3}$从而保证输入到3x3的输入通道数减小。SqueezeNet的网络结构由若干个 fire module 组成

**速度考量**
SqueezeNet在网络结构中大量采用1x1和3x3卷积核是有利于速度的提升的，对于类似caffe这样的深度学习框架，在卷积层的前向计算中，采用1x1卷积核可避免额外的im2col操作，而直接利用gemm进行矩阵加速运算，因此对速度的优化是有一定的作用的。




(4) mobilenet

mobilenet也是用卷积拆分的方法 

![mobilenet][4]


作出更多共享的是有一个width Multiplier宽度参数和resolution Multiplier 分辨率参数 ，可以降低更多的参数。

没使用这两个参数的mobilenet是VGGNet的1/30.
![mobilenet][5]


# 权重参数量化与剪枝
主要是通过权重剪枝，量化编码等方法来实现模型压缩。deep compression是float到uint的压缩,Binarized Neural Networks是 uint 到bool的压缩。

一些技巧：
1. 网络修剪
采用当网络权重非常小的时候(小于某个设定的阈值),把它置0,就像二值网络一般；然后屏蔽被设置为0的权重更新，继续进行训练；以此循环，每隔训练几轮过后，继续进行修剪。

2. 权重共享
对于每一层的参数,我们进行k-means聚类,进行量化,对于归属于同一个聚类中心的权重,采用共享一个权重,进行重新训练.需要注意的是这个权重共享并不是层之间的权重共享,这是对于每一层的单独共享

3. 增加L2权重
增加L2权重可以让更多的权重，靠近0，这样每次修剪的比例大大增加。


## DeepCompresion

![deep compression pipeline][6]

文章早期的工作，是Network Pruning，就是去除网络中权重低于一定阈值的参数后，重新**finetune一个稀疏网络**。在这篇文章中，则进一步添加了量化和编码，思路很清晰简单如下。

### 网络剪枝：移除不重要的连接
（1） 普通网络训练；

（2） 删除权重小于一定阈值的连接得到**稀疏网络**；

（3） 对稀疏网络再训练；

### 权重量化与共享

此处的权值量化基于**权值聚类**，将连续分布的权值**离散化**，从而减小需要存储的权值数量。

让许多连接共享同一权重，使原始存储整个网络权重变为只需要存储码本(有效的权重)和索引；

对于一个4×4的权值矩阵，量化权重为4阶（-1.0，0，1.5，2.0）。

![量化过程][7]

对weights采用**cluster index进行存储**后，原来需要16个32bit float，现在只需要4个32bit float码字，与16个2bit uint索引，参数量为原来的(16×2+4×32)/(16×32)=0.31。

存储是没问题了，那如何对量化值进行更新呢？事实上，文中仅对码字进行更新。如上图：**将索引相同的地方梯度求和乘以学习率**，叠加到码字。

这样的效果，就等价于不断求取weights的聚类中心。原来有成千上万个weights，现在经过一个有效的聚类后，每一个weights都用其**聚类中心进行替代**.

看下表就知道最终的压缩效率非常可观，把500M的VGG压缩到到了11M，1/50。


![压缩VGG][8]

### 霍夫曼编码：更高效利用了权重的有偏分布；


## Binarized Neural Networks

- 提出了一个BWN（Binary-Weight-Network）和XNOR-Network，前者只对网络参数做二值化，带来约32x的存储压缩和2x的速度提升，而后者对网络输入和参数都做了二值化，在实现32x存储压缩的同时带了58x的速度提升；
- 提出了一个新型二值化权值的算法；
- 第一个在大规模数据集如ImageNet上提交二值化网络结果的工作；
- 无需预训练，可实现training from scratch。



# reference


[CNN 模型压缩与加速算法综述](https://cloud.tencent.com/community/article/678192)

[知乎:为了压榨CNN模型，这几年大家都干了什么](https://zhuanlan.zhihu.com/p/25797790)

[ 深度学习（六十）网络压缩简单总结](http://blog.csdn.net/hjimce/article/details/51564774)

[mobile_net的模型优化](https://dragonfive.github.io/2017-07-17/mobilenets/)


[squeeze_net的模型优化](https://dragonfive.github.io/2017-07-20/squeeze_net/)

[神经网络模型演化](https://dragonfive.github.io/2017-07-05/deep_learning_model/)


  [1]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502693251377.jpg
  [2]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502694686047.jpg
  [3]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1507038803077.jpg
  [4]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502694868072.jpg
  [5]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502695760857.jpg
  [6]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1507039484844.jpg
  [7]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502698285109.jpg
  [8]: https://www.github.com/DragonFive/CVBasicOp/raw/master/1502698400878.jpg