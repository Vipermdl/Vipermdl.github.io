---
layout: post
title: "[paper]Self-Ensembling Attention Networks: Addressing Domain Shift for Semantic Segmentation"
excerpt: "The note of Domain Adaptation for Semantic Segmentation"
date: 2019-04-05 15:34:00
mathjax: true
---

***1. Abstract***

采用域适配的原因：these models may not generalize well to unseen image domains due to the phenomenon of domain shift. Since pixel-level annotations are laborious to collect, developing algotithms which can adapt labeled data from source domain to target is great significance.

主要贡献点：

1.首次在域适配任务中引入self-ensemble 迁移学习机制来做语义分割任务

2.在模型中使用注意力机制

3.最重要的是[代码已经开源](https://github.com/YonghaoXu/SEANet)

之所以使用self-ensemble 机制的原因：The initial inspiration of our work comes from an observation that a human can generally make a consistent inter pretation without effort when an image changes slightly. We believe that a good model would possess this characteristic just like humans do. In the light of this statement, an ensemble of multiple networks is a better model than a single network, since the ensemble model generally yields better predictions and shows more robustness towards noise.

self-ensemble 主要结构： 主要分为student network 和 teacher network。 The student network is jointly optimized with supervised segmentation loss from the source domain and the unsupervised consistency loss from the target domain. The teacher network does not participate in the
back-propagation, and is updated with an exponential moving average method using the parameters in the student network at different training time steps. In the test phase, the target-domain images are sent to the teacher network to obtain domain-invariant segmentation maps。

***2. Method***

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-05/SEANet.jpg" width="750"/> 
</div>


论文采用预训练的VGG16为backbone的DeepLav-v2算法（作为AAAI 18 貌似分割模型有点老，投来一点点小小的鄙视眼光，但不影响整体的崇拜！）。在数据增强方面，增加高斯噪声增加模型的泛化性能。原域的图片只喂到学生网络进行分割任务的训练，目标域的图片同时喂到教师网络以及学生网络，然后得到consistency loss。学生网络通过结合两个损失函数（segmentation loss 和 consistency loss）进行学习，教师网络利用学生网络的参数通过指数平滑（线性加权的一种方式进行更新参数）。

有必要说一下consistency loss 以及指数平滑的方式:

本文采用均方差（MSE），其他的文献中也可能采用交叉熵等损失函数，作为约束原域和目标域输出结果的一种手段。在本文中

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-05/algorithm.jpg"/> 
</div>

***3. Performance Evaluation***

<div style="color:#0000FF" align="center">
<img src="/image/2019-04-05/result.jpg" width="750"/> 
</div>


***4. Reference***

1.[Self-Ensembling Attention Networks: Addressing Domain Shift for Semantic Segmentation](https://www.aaai.org/Papers/AAAI/2019/AAAI-XuYonghao.4477.pdf)

2.[Maximum Classifier Discrepancy for Unsupervised Domain Adaptation](http://proceedings.mlr.press/v80/hoffman18a/hoffman18a.pdf)

3.[CyCADA: Cycle-Consistent Adversarial Domain Adaptation](https://arxiv.org/pdf/1712.02560.pdf)