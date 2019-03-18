---
layout: post
title: "[DL]CVPR2019 论文阅读笔记(1)"
excerpt: "针对ScratchDet: Training Single-Shot Object Detectors from Scratch（cvpr2019)进行总结以及整理"
date: 2019-03-18 13:34:00
mathjax: true
---

**1.Preface**

期间看了很多论文，没有整理，加上申博以及论文的事情，焦头烂额。三个多月没有更博，今天重新拾起。将最近阅读的paper进行整理。

**2.Introduction**

当前现有的检测训练任务存在三个限制：

（1）分类任务于检测任务的learning objective bias:一方面事两者任务的不同，检测主要趋向于细节以及语义信息，分类主要趋向于全局信息，造成两者在网络训练时损失函数的不同；一方面是两者对平移不变性的敏感度不同。

（2）关于迁移学习的问题，数据集中的不同以及任务的不同。比如说，ImageNet 数据集是单图单物体，而coco && pascol数据集是单图多物体。从自然生活场景图片迁移到医疗图像中，卫星检测。不同域、不同数据集中是否能发挥作用？

（3）在目标检测任务中，通常使用image-net预训练网络进行初始化网络然后进行fune-tine.在更改网络结构后，采用image-net预训练网络代价较大；另一方面，在嵌入式设备运行小模型时，image-net预训练网络的时间于计算资源消耗都比较大。因此，采用image-net预训练网络并非真正的现实可行。

**3.Analysis**

因此，论文首先讨论随机初始化训练的DSOD将必要条件归结到一阶段检测器和DenseNet的dense layer-wise connnection上，但是这样做很大程度上限制了网络结构的设计，而作者通过对比实验，将DSOD中得BN层去掉， mAP从77.7%下降到71.5%。因此，作者认为BN才是从头训练DSOD成功的关键。

作者受到NIPS2018《How Does Batch Normalization Help Optimization?》文章的启发，通过理论和实验结果说明BN在优化过程中发挥的作用：

（1）梯度更加稳定，更加可预测。

（2）计算梯度时可采用更大的步长，即更大的学习率来加速训练。

（3）防止loss函数解空间突变，既不会掉入梯度消失的平坦区域，也不会调入梯度爆照的局部最小。

沿着这个思路作者在SSD300检测框架上给VGG网络与检测子网络分别加上了BN来进行随机初始化训练（PASCAL VOC 07+12训练，07测试），调整学习率之后，得到的最好结果78.7%mAP，比直接随机初始化训练SSD的结果（67.6%）高11.6%，比原SSD300（77.2%）高1.5%，比使用预训练模型VGG-16-BN（78.2%）高0.6%。实验细节写在论文里。

接着随机初始化训练以及BN带来的优势，我们可以肆无忌惮的对特征提取网络进行改动。

因此作者借鉴了ResNet与VGGNet在目标检测与分类任务之间的优缺点（具体可看[DSSD](https://arxiv.org/abs/1701.06659)），首先把ResNet的第一个卷积层的stride从2改成1，也就是取消第一个下采样操作， 并且参照了DSOD的方法，替换第一个卷积层为3个3x3卷积层：这样做的目的是，尽可能保持原图信息不损失，并且充分利用。

注意：在将新网络替换到SSD框架上时，仍然最大程度保证实验的公平性。首先，用于检测的特征图在论文中保持38×38, 19×19, 10×10, 5×5, 3×3, 1×1的大小，并没有使用大的特征图；其次，保证每个用于检测的特征图的channel数目相同。

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-18/root-resNet-18.png" width="860"/> 
</div>

(a) 原ResNet-18: 结果为73.1% mAP。

(b) ResNet-18-A: 去掉了ResNet-18的max-pooling层，即取消第二个下采样操作，结果为75.3% mAP。

(c) ResNet-18-B: 将ResNet-18的第一个卷积层的stride=2改为1，即取消第一个下采样操作，结果为77.6% mAP 。

(d) Root-ResNet-18: 将ResNet-18-B的第一个7x7卷积核替换成3个3x3卷积，结果为78.5% mAP。

分析：在300x300大小的输入图像上（小物体较多）：

对比(a)与(c): 取消第一个下采样操作，提升了4.5% mAP。

对比(a)与(b): 取消第二个，保留第一个下采样操作 , 提升2.2% mAP。

对比(b)与(c): 是否对原图进行下采样，会有2.3% mAP的影响。

对比(c)与(d): 替换7x7为3个3x3，使用更加冗余的特征会提升结果。

另外，作者将SSD在特征提取网络的各级特征预测模块替换为残差模块进行预测，减少了参数量，计算量，提升了FPS（SSD300-Root-Res34:20FPS->25FPS）, 而且检测准确率没有下降（在VOC07上，80.4% mAP)。

**4.Conclusion**

在kaiming的[rethink](https://arxiv.org/abs/1811.08883)里面，也强调了使用Normalization的重要性，因此，我认为关于随机初始化训练检测器的必要条件为：

（1）需要稳定梯度的优化手段（clip_gradient、BN、GN、SN等）。

（2）训练足够多的epoch与合适的学习率。

（3）数据集的规模（数据集越大越好）。

（4）随机初始化训练需要更大的步长来进行全局最优搜索。

**5.Reference**

1.[DSSD : Deconvolutional Single Shot Detector](https://arxiv.org/abs/1701.06659)

2.[Rethinking ImageNet Pre-training](https://arxiv.org/abs/1811.08883)

3.[ScratchDet: Training Single-Shot Object Detectors from Scratch](https://arxiv.org/abs/1810.08425)