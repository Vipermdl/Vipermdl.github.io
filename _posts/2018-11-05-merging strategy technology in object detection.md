---
layout: post
title: "[DL]目标检测算法中检测框合并策略技术"
excerpt: "参考sigai平台的检测框算法综述，针对nms、soft-nms、softer-nms进行总结。"
date: 2018-11-05 13:34:00
mathjax: true
---

**前言：**

在目标检测算法中，常常利用非极大值抑制算法（NMS, non maximum suppression）对生成的大量候选框进行后处理，去除冗余的候选框，得到最佳检测框，以加快目标检测的效率。其本质思想是搜索局部最大值，抑制非极大值。本文参考参考sigai平台的检测框算法[综述](../cellar/merging strategy technology in object detection algorithm.pdf)，针对nms、soft-nms、softer-nms等常用算法进行总结。


**1.NMS**

NMS主要就是通过迭代的形式，不断的以最大得分的框去与其他框做IOU操作，并过滤那些IOU较大的框，具体计算过程如下：

1.根据候选框的类别分类概率做排序，假如有4个BBox,其置信度A>B>C>D.
2.先标记最大概率矩形框A是算法要保留的BBox;
3.从最大概率矩形框A开始，分别判断A、B、C与D的重叠度IOU是否大于某个设定的阈值（0.5），假设D与A的重叠度超过阈值，那么就舍弃D；
4.从剩下的举行框B、C中选择概率最大的B，标记为保留，然后判读C与B的重叠度，扔掉重叠度超过设定阈值的矩形框；
5.一直重复进行，标价完所有要保留下来的矩形框。

NMS缺点:

NMS算法中的最大问题就是它将相邻检测框的分数均强制归零（即将重叠部分大于重叠阈值Nt的检测框移除）。在这种情况下，如果一个真实物体在重叠区域中，则将导致对该物体的检测失败并降低了算法的平均检测率。

NMS在检测是，将所有类别的分类置信度进行排列，而强制归零则会使特定场景中的小目标、重叠目标的检测变差。NMS只能作为局部搜索中的最优解使用。同时，NMS的阈值不太容易确认，设置过大会出现误删，设置过高又容易增大误检。从RCNN到Fast-rcnn到Faster-rcnn，阈值均设置为0.5.但针对其他数据集，重叠阈值应当调整。

**2.Soft-NMS**

这篇是ICCV2017的文章，是针对NMS算法缺点的改进，论文题目很霸气：一行代码改进目标检测，即《Improving Object Detection With One Line of Code》由 UMIACS 大学提出。

NMS算法略显粗暴，因为NMS直接将射出所有IOU大于阈值的框。Soft-NMS在算法执行过程中不是简单的对IOU大于阈值的检测框进行删除，而是降低得分。算法基本流程同NMS相同，但是对原置信度得分使用函数运算，目标是降低置信度得分。

经典的NMS算法将IOU大于阈值的窗口的得分全部置为0，可表述如下：

<div align="center"><img src="/image/2018-11-05/nms.png" /></div>

论文置信度重置函数有两种形式改进，一种是线性加权：

<div align="center"><img src="/image/2018-11-05/soft-nms1.png" /></div>

一种是高斯加权的：

<div align="center"><img src="/image/2018-11-05/soft-nms2.png" /></div>

soft-NMS缺点：

1.soft-NMS在训练中采用传统的NMS方法，仅在推断代码中实现soft-NMS。据此，可以推断作者应该做过对比实验，在训练过程中采用soft-NMS没有显著提高。

2.NMS是soft-NMS特殊形式，当得分重置函数f采用二值化函数时，soft-NMS和NMS是相同的。soft-NMS算法是一种更加通用的非最大抑制算法。

综上，soft-NMS在动机上并不难理解，同时实验在不同的检测算法中都有将近0.02的mAP效果提升。实现较为简单，但在缺点1中显示，在训练中soft-nms并没有什么用。提升效果不大。因此，相比NMS的改进较少，同时contribution较少。

**3.Softer-NMS**

近期，卡内基梅隆大学与旷视科技的研究人员提出了一种新的非极大抑制算法Softer-NMS，其方法是在Soft-NMS和NMS基础上改进，论文不是简单的更改CNN结构或调整参数，引入高斯分布， 狄拉克delta分布， KL 散度等数学知识。总体上数学推导完美，但是我感觉论文的目的性不强，并没有很好的解释原因，与NMS出入较大，最后的NMS更可以将其作为相同候选框的取平均操作。但为NMS操作提供了新的思路，值得一读，哈哈！具体contribution如下：

改进的<a href="http://www.codecogs.com/eqnedit.php?latex=loss_{regression}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?loss_{regression}" title="loss_{regression}" /></a>：

注意，算法在坐标预测时，由之前算法的<a href="http://www.codecogs.com/eqnedit.php?latex=(x,&space;y,&space;w,&space;h)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?(x,&space;y,&space;w,&space;h)" title="(x, y, w, h)" /></a>变为<a href="http://www.codecogs.com/eqnedit.php?latex=(x_{min},&space;y_{min},&space;x_{max},&space;y_{max})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?(x_{min},&space;y_{min},&space;x_{max},&space;y_{max})" title="(x_{min}, y_{min}, x_{max}, y_{max})" /></a>，其坐标变换的形式与之前算法相同。

算法在网络的坐标预测层增加一分支，与fc7层相同，但采用绝对值代替ReLU,使得网络能输出<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>作为BBox的标准差做NMS操作。

When <a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>作为BBox的<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>->0, it means our network is extremely confident about estimated bounding box location.

因此，我们将坐标概率化，呈正态分布。其中预测坐标的概率分布为：

<div align="center"><img src="/image/2018-11-05/estimated coordinate.png" /></div>

GroundTruth坐标的概率分布为：

<div align="center"><img src="/image/2018-11-05/GT coordinate.png" /></div>

<a href="http://www.codecogs.com/eqnedit.php?latex=x_{e}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?x_{e}" title="x_{e}" /></a>为预测坐标，<a href="http://www.codecogs.com/eqnedit.php?latex=x_{g}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?x_{g}" title="x_{g}" /></a>为groundtruth坐标。

因此，我们检测的目标就是对两者概率进行KL-Divergence:

<div align="center"><img src="/image/2018-11-05/loss.png" /></div>

针对<a href="http://www.codecogs.com/eqnedit.php?latex=x_{e}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?x_{e}" title="x_{e}" /></a>和<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>进行优化。论文中详细阐述了求导过程、改进以及与之前算法<a href="http://www.codecogs.com/eqnedit.php?latex=loss_{regression}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?loss_{regression}" title="loss_{regression}" /></a>的联系。


softer-NMS流程图：

<div align="center"><img src="/image/2018-11-05/softer-nms.png" /></div>

当两个BBox之间的重叠度IOU是否大于某个设定的阈值，针对两BBOX做合并操作，而不是像之前的NMS（去掉）softer-NMS(降低得分)。具体操作如下：

<div align="center"><img src="/image/2018-11-05/avg-softer.png" /></div>


softer-NMS缺点分析：

1.首先肯定作者从概率论角度针对<a href="http://www.codecogs.com/eqnedit.php?latex=loss_{regression}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?loss_{regression}" title="loss_{regression}" /></a>进行推导，进而改变RCNN网络结构，增加<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>变量，方便后续的Softer-nms操作。

2.论文提出的Softer-NMS,从本质上来说其实时对预测的检测框加权取平均，为什么要这么做？以及网络中输出的<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>为什么就是我们数学推导中所需要的<a href="http://www.codecogs.com/eqnedit.php?latex=\sigma" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\sigma" title="\sigma" /></a>，并没有很好的理论依据。只是从实验上显示，softer-nms可以提升效果。

**IOU-Net**

IoU-Net 是旷视的另外一篇论文，是 ECCV2018 接收并做口头报告（清华和北大的学生在旷视实习时完成提交），和 Softer-NMS 一样，基于CNN的目标检测方法存在的分类置信度和定位置信度不匹配的问题。

我曾在之前的[博客](https://vipermdl.github.io/2018/08/14/IOU-Net/)中有所提及，在此不做过多阐述。


**总结**

在实际应用中，我还是比较倾向于使用传统的NMS操作。虽然NMS缺点较多，但是基于现在的该进并不是很多，提升效果不是很大，期待更多的大佬贡献好的trick。

**参考文献**

[1] https://github.com/hoya012/deep_learning_object_detection

[2] Bodla N, Singh B, Chellappa R, et al. Soft-nms—improving object detection with one line of code[C]//Computer Vision (ICCV), 2017 IEEE International Conference on. IEEE, 2017: 5562-5570.

[3] He Y, Zhang X, Savvides M, et al. Softer-NMS: Rethinking Bounding Box Regression for Accurate Object Detection[J]. arXiv preprint arXiv:1809.08545, 2018.

[4] Borui Jiang, Ruixuan Luo,et al. Acquisition of Localization Confidence for Accurate Object Detection. arXiv preprint arXiv: 1807.11590, 2018.

[5] Han Hu, Jiayuan Gu,Zheng Zhang,et al.Relation Networks for Object Detection. arXiv preprint arXiv: 1711.11575, 2017












