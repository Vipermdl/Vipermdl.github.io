---
layout: post
title: "[paper]Rethinking the paper of Deformable ConvNet v1 vs v2"
excerpt: "Deformable ConvNets v2: More Deformable, Better Results 论文思考"
date: 2018-12-08 13:34:00
mathjax: true
---

**前言：**

本文仅仅阐述针对可变形卷积的几点思考，不做基础知识的普及：

	1. 关于 Deformable ConvNetsv1 推荐阅读[博客](https://medium.com/@phelixlau/notes-on-deformable-convolutional-networks-baaabbc11cf3)。
	
	2. Deformable ConvNetsv2 在v1的基础上，增加了权重。 并将其应用在conv3~conv5层（v1仅仅应用在conv5层）。

	3. 提出 R-CNN Feature Mimicking， 使得 rpn 与fast rcnn head 层联系更加密切。

**心得：**

	1. 模型可视化动机和前期分析，可视化结果。

	2. appandix提及多尺度test，可以做到小物体涨点，同时其他物体由于可变形卷积的存在做到不掉点。

	3. 似乎可变形pooling的存在感不是很强，在v1中的涨点也不是很多。

	4. eature mimicking对于regular conv不怎么涨这一点让人有点在意（paper里claim是因为regular conv的capacity有限），作为这篇paper里有novelty的一个part来说，文中仅仅用了一句话阐述原因，值得怀疑。

	5. 能否将其作为比赛的一个trick还是有待商榷，过段时间跑个deeplabv3+ + deformable conv2 做个实验尝试一下。

**参考文献**

[1] [Deformable ConvNets v2: More Deformable, Better Results](https://arxiv.org/abs/1811.11168)

[2] [Deformable Convolutional Networks](https://arxiv.org/abs/1703.06211)

[3] [如何评价 MSRA 最新的 Deformable Convolutional Networks](https://www.zhihu.com/question/57493889)

[4] [PyTorch implementation of Deformable Convolution](https://github.com/oeway/pytorch-deform-conv)

[5] [Deformable Convolutional Networks V2 with Pytorch](https://github.com/CharlesShang/DCNv2)












