---
layout: post
title: "[Face detector]SSD算法在人脸检测领域的使用总结"
excerpt: "针对S3FD、FaceBoxes三篇人脸检测论文进行总结以及整理"
date: 2019-03-19 15:34:00
mathjax: true
---

**Introduction**

SSD主要有以下几个主要特点:

1. 特征提取主干网络：VGG16，去除全连接层fc8，fc6 和 fc7层转换为卷积层，pool5不进行分辨率减小，在fc6上使用dilated convolution弥补损失的感受野；并且增加了一些分辨率递减的卷积层；

2. SSD摈弃了proposal的生成阶段，使用anchor机制，这里的anchor就是位置和大小固定的box，可以理解成事先设置好的固定的proposal

3. SSD使用不同深度的卷积层预测不同大小的目标，对于小目标使用分辨率较大的较低层，即在低层特征图上设置较小的anchor，高层的特征图上设置较大anchor

4. 预测模块：使用3x3的卷积对每个anchor的类别和位置直接进行回归

5. SSD使用的data augmentation对效果影响很大

当前，SSD的算法的鲁棒性较好，因此在工业界应用相对较多，同时基于SSD算法的改进也层出不穷，之前的cvpr2019中的ScratchDet就是在SSD上的改进，推荐一篇[SSD系列论文总结](https://zhuanlan.zhihu.com/p/35642094),总结的很棒。

然而，SSD算法并不能直接应用于人脸检测领域。SSD算法受小目标检测性能较弱的影响，直接应用SSD算法并不能很好的解决人脸检测领域中小人脸问题。我曾使用SSD算法在WIDER Face数据集上进行训练，在FIDD数据集上测试，结果如下图所示：

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-04/x.jpg" width="800" height="400"/> 
</div>

由此，我们简要总结针对SSD算法在人脸领域改进的两篇文章：S3FD、FaceBoxes。


**FaceBoxes: A CPU Real-time Face Detector with High Accuracy**

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-04/faceboxes.jpg" width="800"/> 
</div>

主要贡献点：

1. Rapidly Digested Convolutional Layers(RDCL)

（1）在网络前期，使用RDCL快速缩小feature map的大小。conv1,pool1,conv2,pool2的strdie分别是4，2，2和2。能够快速减小feature map大小，实现人脸检测的实时性。

（2）卷积（或pooling）核太大速度就慢，太小覆盖信息又不足。文章权衡之后，将conv1,pool1,conv2,pool2的核大小分别设为7*7，3*3，5*5，3*3。

（3）使用CReLU来保证输出维度不变的情况下，减少卷积核的数量。

2. Multiple Scale Convolutional Layers(MSCL)

在网络后期，使用MSCL更好地检测不同尺度的人脸。主要设计原则有：

（1）类似于SSD，在网络的不同层进行检测；

（2）采用inception模块。由于inception包含多个不同的卷积分支，因此可以进一步使得感受野多样化。

3. Anchor densification strategy

针对SSD对小目标检测性能不好的原因，论文在Inception3 以及conv3_2两个检测分支分别增加anchor的密度，分别增加4倍和两倍。具体操作可以看代码，实现较为简单。

**S3FD: Single Shot Scale-invariant Face Detector**

这篇ICCV2017关于人脸检测的文章正是为了解决小尺寸人脸难以检测的问题。这篇文章的出发点是：当人脸尺寸比较小时，基于anchor的人脸检测算法效果下降明显，因此提出了不受人脸尺度变化影响的S3FD算法。具体内容如下：

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-04/s3fd.jpg"/> 
</div>

1. 原因分析：

（1）中展示的是网络结构本身的设计问题。我们知道在SSD算法中有多个特征层用于检测目标，这些特征层中stride最小的是8，这样原图中8x8大小的区域在该预测层中就仅有1个像素点，这对于小人脸的检测是非常不利的，因为有效的特征太少了。同样，在Faster RCNN算法中，用于检测目标的特征层的stride是16，用于小人脸检测的有效特征区域更小了。

（2）中展示的是anchor尺寸、感受野和人脸尺寸不匹配的问题。从图中可以看出anchor尺寸和感受野大小不是很匹配，同时这两个都远大于小人脸。

（3）中因为一般设置的anchor尺寸都是离散的，比如[16,32,64,128,256,512]，但是人脸的尺寸是连续的，因此当人脸尺寸在设定的anchor值之间时能够用于检测的anchor数量就很少，如图中红色圆圈表示，这样就容易导致人脸检测的召回率降低。

（4）为了提高小人脸的检测召回，很多检测算法都会通过设置较多的小尺寸anchor实现，这样容易导致较多的小尺寸负样本anchor，最终导致误检率的上升。例如（d）中两张图的分辨率一样，左边图中人脸区域较小，因此主要通过浅层特征进行检测，此时anchor尺寸设置较小；右图中人脸区域较大，因此主要是通过高层特征进行检测，此时anchor尺寸设置较大。可以看出左图中标签为背景的anchor数量远远多于标签为目标的anchor，而在右图中这种现象相对好一些。

2. 改进内容主要包括：

（1）改进检测网络和设置更加合理的anchor，改进检测网络主要是增加stride=4的预测层，保证小人脸在浅层进行检测时能够有足够的特征信息。anchor的尺寸根据预测层的有效感受野和等比例间隔原理进行设置，保证每个预测层的anchor尺寸能参考有效感受野。

（2）引入尺度补偿的anchor匹配策略增加正样本anchor的数量，这部分主要分两步，第一步还是和常规确定anchor的正负标签类似，只不过将IOU阈值从0.5降到0.35，也就是和ground truth的IOU大于0.35的anchor为正样本，这样就先保证每个目标都能有足够的anhcor来检测，这样相当于间接解决了原本处于不同anchor尺寸之间的人脸的可用anchor数量少的问题。第一步之后，仍然会有较多的小人脸没有足够的正样本anchor来检测，因此第二步的目的就是提高小人脸的正样本anchor数量，具体而言是对所有和ground truth的IOU大于0.1的anchor做排序，选择前N个anchor作为正样本，这个N是第一步的anchor数量均值，提高人脸的召回。

（3）引入max-out background label降低误检，摘自论文原话：“For each of the smallest anchors, we predict Nm scores for background label and then choose the highest as its final score, as illustrated in Fig. 4(b). Max-out operation integrates some local optimal solutions into our S3FD model so as to reduce the false positive rate of small faces.”。 通过引入不平衡样本分类方式来降低误检。


**Reference**

1.[SSD：single shot multibox detector](https://arxiv.org/abs/1512.02325)

2.[FaceBoxes: A CPU Real-time Face Detector with High Accuracy](http://cn.arxiv.org/abs/1708.05234)

3.[S3FD: Single Shot Scale-invariant Face Detector](https://arxiv.org/abs/1708.05237)