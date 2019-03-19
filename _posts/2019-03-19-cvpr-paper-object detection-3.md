---
layout: post
title: "[DL]CVPR2019 论文阅读笔记(3)"
excerpt: "针对cvpr2019中mask-rcnn的两篇改进版mask scoring rcnn 和 HTC 进行总结以及整理"
date: 2019-03-19 15:34:00
mathjax: true
---

**1.Mask Scoring R-CNN**

这篇论文从实力分割中mask的分割质量角度出发，提出mask实力分割框架中存在的一个缺陷，用Bounding box的分类置信度作为mask score,导致mask score 和mask quality 不配准。因此文章基于mask rcnn提出一个新的框架Mask Scoring R-CNN, 能自动学习出mask quality, 试图解决不配准的问题。（就本文而言，我感觉思路与IOU-Net相似，均是找任务之间的gap,并针对gap做改进）。

在Mask-RCNN中，mask分支的分割质量来源于检测分支的classification confidence. Mask R-CNN其实是Faster R-cnn系列的延伸，其在Faster R-CNN的基础上添加一个新的分支用来预测object mask，该分支以检测分支的输出作为输入，mask的质量一定程度上依赖于检测分支。这种简单粗暴的做法取到了SOTA的性能。在本文中，作者在Mask R-CNN的基础上添加一个MASK IOU分支用于预测当前输出的mask和gt mask的IOU。MaskIOU的输入由两部分组成，一是ROI Align得到的ROI feature map,二是mask分支输出的mask,两者concat之后经过3层卷积核2层全连接输出MaskIOU.

training过程：
box分支和mask保持不变，输出的mask先经过阈值为0.5的binarize，再计算binary mask和gt的IoU作为target，采用L2 loss作为损失函数，loss weight设为1，3个分支同时end-to-end训练。

inference过程：
检测分支输出score最高的100个框，再送入mask分支，得到mask结果，RoI feature map再和mask送入MaskIoU分支得到mask iou，与box的classification score相乘就得到最后的mask score。

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-19/MS RCNN.png" width="800" height="400"/> 
</div>

总结：
作者motivation就是想让mask的分数更合理，从而基于mask rcnn添加一个新的分支预测来得到更准确的分数，做法简单粗暴，从结果来看也有涨点。其实mask的分割质量也跟box输出结果有很大关系，这种detection-based分割方法不可避免，除非把detection结果做的非常高，不然mask也要受制于box的结果。这种做法与IoU-Net类似，都是希望直接学习最本质的metric方式来提升性能。

**2.Hybrid Task Cascade for Instance Segmentation**

这篇论文提出了一种新的实例分割框架，设计了多任务多阶段的混合级联结构，并且融合了一个语义分割的分支来增强 spatial context。效果明显优于 Mask R-CNN 和 Cascade Mask R-CNN 的结果。

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-19/HTC.png" width="820" height="400"/> 
</div>

（1）由于 Cascade R-CNN 在物体检测上的结果非常好，论文首先尝试将 Cascade R-CNN 和 Mask R-CNN 直接进行杂交，得到子代 Cascade Mask R-CNN，如上图（a）所示。在这种实现里，每一个 stage 和 Mask R-CNN 相似，都有一个 mask 分支 和 box 分支。当前 stage 会接受 RPN 或者 上一个 stage 回归过的框作为输入，然后预测新的框和 mask。但是该种设计存在明显的问题，实验结果表明，主要在于 Cascade Mask R-CNN 相比 Mask R-CNN 在 box AP 上提高了 3.5 个点，但是在 mask AP 上只提高了 1.2 个点。

（2）Cascade R-CNN 虽然强行在每一个 stage 里面塞下了两个分支，但是这两个分支之间在训练过程中没有任何交互，它们是并行执行的。所以我们提出 Interleaved Execution，也即在每个 stage 里，先执行 box 分支，将回归过的框再交由 mask 分支来预测 mask，如上图（b）所示。这样既增加了每个 stage 内不同分支之间的交互，也消除了训练和测试流程的 gap。实验发现这种设计对 Mask R-CNN 和 Cascade Mask R-CNN 的 mask 分支都有一定提升。

（3）这一步起到了很重要的作用，对一般 cascade 结构的设计和改进也具有借鉴意义。我们首先回顾原始 Cascade R-CNN 的结构，每个 stage 只有 box 分支。当前 stage 对下一 stage 产生影响的途径有两条：（1）B_{i+1} 的输入特征是 B_i 预测出回归后的框通 RoI Align 获得的；（2） B_{i+1} 的回归目标是依赖 B_i 的框的预测的。这就是 box 分支的信息流，让下一个 stage 的特征和学习目标和当前 stage 有关。在 cascade 的结构中这种信息流是很重要的，让不同 stage 之间在逐渐调整而不是类似于一种 ensemble。
然而在 Cascade Mask R-CNN 中，不同 stage 之间的 mask 分支是没有任何直接的信息流的， M_{i+1} 只和当前 B_i 通过 RoI Align 有关联而与 M_i 没有任何联系。多个 stage 的 mask 分支更像用不同分布的数据进行训练然后在测试的时候进行 ensemble，而没有起到 stage 间逐渐调整和增强的作用。为了解决这一问题，我们在相邻的 stage 的 mask 分支之间增加一条连接，提供 mask 分支的信息流，让 M_{i+1} 能知道 M_i 的特征。我们将 M_i 的特征经过一个 1x1 的卷积做 feature embedding，然后输入到 M_{i+1} ，这样 M_{i+1} 既能得到 backbone 的特征，也能得到上一个 stage 的特征.

（4）尝试将语义分割引入到实例分割框架中，以获得更好的 spatial context。因为语义分割需要对全图进行精细的像素级的分类，所以它的特征是具有很强的空间位置信息，同时对前景和背景有很强的辨别能力。通过将这个分支的语义信息再融合到 box 和 mask 分支中，这两个分支的性能可以得到较大提升。在具体设计上，为了最大限度和实例分割模型复用 backbone，减少额外参数，论文在原始的FPN 的基础上增加了一个简单的全卷积网络用来做语义分割。首先将 FPN 的 5 个 level 的特征图 resize到相同大小并相加，然后经过一系列卷积，再分别预测出语义分割结果和语义分割特征。使用 COCO-Stuff 的标注来监督语义分割分支的训练。红色的特征将和原来的 box 和 mask 分支进行融合（在下图中没有画出），融合的方法我们也是采用简单的相加。

最后，实验给出最终结果的涨点技巧对比，如下图所示。确实是一篇很好的paper。值得大家去阅读代码。待我阅读完代码再来吹逼！

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-19/results.png"/> 
</div>

**3.Reference**

1.[实例分割的进阶三级跳：从 Mask R-CNN 到 Hybrid Task Cascade](https://zhuanlan.zhihu.com/p/57629509)

2.[Hybrid Task Cascade for Instance Segmentation](https://arxiv.org/abs/1901.07518)

3.[Mask Scoring R-CNN](https://arxiv.org/abs/1903.00241)