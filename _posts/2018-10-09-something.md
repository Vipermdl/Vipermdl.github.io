---
layout: post
title: "[papers]关于目标检测感兴趣领域整理"
excerpt: "针对前段时间遗留论文的理解、整理以及想法"
date: 2018-10-09 14:39:00
mathjax: true
---

--**前言**--

当前，大量的学术机构包括工业界学者仍然将注意力放在目标检测上，并将其应用在无人驾驶、智能监控、机器人、行人及人脸识别，商标识别等领域。基于深度学习的目标检测技术，在网络结构上，从two-stage到one-stage, 从single scale network 到 feature pyramid network, 通过不过的改进，使得目标检测算法在数据集上的检测效果和性能都表现出色。R-CNN、 R-FCN、 Fast/Faster R-CNN、 SSD、 以及YOLO 系列, 都在目标检测领域具有里程碑式的意义。个人认为，18年之后提出的目标检测算法并没有像以上算法提出时那般轰动。强如今年cvpr2018 Cascade R-CNN、 SNIP等，也仅仅是针对之前算法的缺点做微小改动，这似乎预示着，目标检测领域已经越发成熟，研究方向也更为详细。因此，我们应针对特定的应用，特定的检测目标，设计特定的检测算法。我将我感兴趣的领域分为三类，分别为：scale variance、 rotational variance 和 domain adversarial for detection。 并对其进行阐述。

--**scale variance**--

在cvpr2018的SNIP、ECCV2018中的DetNet等论文中都阐述了目标尺度大小对检测算法性能的影响。检测的数据集中目标大小分布差别较大，采用较大的感受野往往会难以检测数据集中的小目标。一方面，我们可以通过图像金字塔方式将图片进行scale,进而能有效的检测出数据集中过大或过小的目标，同时，我们应通过数据增强增加样本的多样性。另一方面，采用与SSD相同的思路，提取多层feature map进行bounding box预测，或通过Dilated Convolutions来增加感受野的尺寸。因此，在目标检测领域，针对特定数据集，构建模型时需要正确反映一些对不同检测目标通用的先验知识，从数据中发掘背后的解释性因子。个人认为，可将表示学习与目标检测相结合，从中找到某些先验知识作为目标检测的评价准则。

--**rotational variance**--

在众多的数据集中，某些数据集中的目标不一定以水平的方式出现，例如IC15、IC13、Dota等。另外，CNN并不能像IC15中出现的带有旋转角度的图片。现阶段的一些工作例如R2CNN、RRPN、textspotter等算法通过改进现阶段的检测算法，使得算法能检测出带有旋转角度的bounding box。他们在某种程度上为检测带有旋转角度图片的目标问题上提供了建设性思路。另一方面，Zheng Zhang在2016年提出的Multi-oriented text detection with fully convolutional networks通过分割的方式提出有向的bounding boxes，以及今年AAAI上面由浙江大学提出的PixelLink: Detecting Scene Text via Instance Segmentation，也采用相同的方式提出建议框。我认为，此方法为检测rotational image 提供了新思路，这也是我目前正在做的事情，及通过分割的方式进行rotational目标检测。

--**domain adversarial for detection**--

迁移学习为人工标注提供了可行的解决方案。而其中的domian-adversarial training 的思想更是让我感受到对抗思想的巧妙，从16年的Domain-Adversarial Training of Neural Networks到18年cvpr Domain Adaptive Faster R-CNN, 该领域仍在持续不断的发展（我曾在我们组会上基于此方向做过详细的汇报，https://github.com/Vipermdl/Vipermdl.github.io/blob/master/cellar/Domain-Adaptive.pdf）。毫无疑问，若精度上达到工业界需求，该应用必将极大的方便我们，相信在未来的一段时间，会有更多优秀的基于Domain Adaption 和 Obeject detection 交叉的工作产生。因此，我将继续专注于该问题的探索和研究。
