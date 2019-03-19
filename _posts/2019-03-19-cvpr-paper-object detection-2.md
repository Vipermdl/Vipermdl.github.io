---
layout: post
title: "[DL]CVPR2019 论文阅读笔记(2)"
excerpt: "针对Generalized Intersection over Union: A Metric and A Loss for Bounding Box Regression（cvpr2019)进行总结以及整理"
date: 2019-03-19 13:34:00
mathjax: true
---

**1.Introduction**

IOU是目标检测领域最重要的评价尺度之一，特性是对尺度不敏感，主要判断检测框的重合程度。但对于CNN而言，没有方向信息，无法反馈神经网络的边界回归框应该如何调整，即直接用IOU作为损失函数会出现两个问题：

（1）如果两个框没有相交，根据定义，IOU=0，不能反映两者的距离大小，同时因为loss=0，没有梯度回传，无法进行学习训练。
（2）IOU无法精确的反映两者的重合度大小。如图1所示，IOU的变化无法反馈定位框的重合度和调整方向。针对IOU上述的两个缺点，本文提出一个新的指标generalized IOU(GIOU)如图2所示

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-19/ALGORITHM.png"/> 
</div>

<div style="color:#0000FF" align="center">
<img src="/image/2019-03-19/GIOU.png"/> 
</div>

**2.Conclusion**

GIoU loss 可以替换掉大多数目标检测算法中 bounding box regression，本文选取了 Faster R-CNN、 Mask R-CNN 和 YOLO v3 三个方法验证 GIoU loss 的效果。 可以看出 YOLOv3 在 COCO 数据集有明显优势，但在其他模型下优势不明显，作者也指出了 Faster rcnn 和 mask rcnn 效果不明显的原因是 anchor 很密， 比如 Faster rcnn 2k 个 Anchor Boxes,各种类型覆盖应有尽有，不仅仅是根据 IoU 和 NMS 挑选合适的检测框，而且需要对检测框进行有方向的修订。总体来说，文章的 motivation 比较好，指出用 L1、 L2 作为 regression 损失函数的缺点，以及用直接指标 IoU 作为损失函数的缺陷性，提出新的 metric 来代替 L1、 L2 损失函数，从
而提升 regression 效果，想法简单粗暴，但相比 state-of-art 没有明显的性能优势。


**4.Reference**

1.[Generalized Intersection over Union: A Metric and A Loss for Bounding Box Regression](https://arxiv.org/abs/1902.09630)

2.[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](http://papers.nips.cc/paper/5638-faster-r-cnn-towards-real-time-object-detection-with-region-proposal-networks)

3.[YOLOv3: An Incremental Improvement](https://arxiv.org/abs/1804.02767)

4.[Mask R-CNN](http://openaccess.thecvf.com/content_ICCV_2017/papers/He_Mask_R-CNN_ICCV_2017_paper.pdf)