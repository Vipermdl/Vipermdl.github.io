---
layout: post
title: "[DL]目标检测中样本不平衡问题的前世今生"
excerpt: "围绕CVPR2016，CVPR2017，AAAI2019的三篇文章，OHEM、focal loss、 GHM等，从prob和loss两个角度讨论。"
date: 2018-12-21 13:34:00
mathjax: true
---

**1.Introduction**

我们知道object detection的算法主要可以分为两大类：two-stage detector和one-stage detector。前者是指类似Faster RCNN，RFCN这样需要region proposal的检测算法，这类算法可以达到很高的准确率，但是速度较慢。虽然可以通过减少proposal的数量或降低输入图像的分辨率等方式达到提速，但是速度并没有质的提升。后者是指类似YOLO，SSD这样不需要region proposal，直接回归的检测算法，这类算法速度很快，但是准确率不如前者。在object detection领域，一张图像可能生成成千上万的candidate locations，但是其中只有很少一部分是包含object的，这就带来了类别不均衡。另一方面，针对数据集样本的困难度，我们可以将图片划分为困难图片和容易图片，同样存在着样本不平衡问题（关于样本困难度阐述定义如下，定义引自GHM）
以二分类图片为例，对于候选框<a href="https://www.codecogs.com/eqnedit.php?latex=p\in&space;[0,1]" target="_blank"><img src="https://latex.codecogs.com/gif.latex?p\in&space;[0,1]" title="p\in [0,1]" /></a>为模型的预测概率，<a href="https://www.codecogs.com/eqnedit.php?latex=p&space;^&space;{&space;*&space;}\in&space;\left&space;\{&space;0,&space;\right&space;1\}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?p&space;^&space;{&space;*&space;}\in&space;\left&space;\{&space;0,&space;\right&space;1\}" title="p ^ { * }\in \left \{ 0, \right 1\}" /></a>为样本的真实标签，则二分类loss如下：

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=L&space;_&space;{&space;C&space;E&space;}&space;\left(&space;p&space;,&space;p&space;^&space;{&space;*&space;}&space;\right)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;-&space;\log&space;(&space;p&space;)&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;-&space;\log&space;(&space;1&space;-&space;p&space;)&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?L&space;_&space;{&space;C&space;E&space;}&space;\left(&space;p&space;,&space;p&space;^&space;{&space;*&space;}&space;\right)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;-&space;\log&space;(&space;p&space;)&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;-&space;\log&space;(&space;1&space;-&space;p&space;)&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." title="L _ { C E } \left( p , p ^ { * } \right) = \left\{ \begin{array} { l l } { - \log ( p ) } & { \text { if } p ^ { * } = 1 } \\ { - \log ( 1 - p ) } & { \text { if } p ^ { * } = 0 } \end{array} \right." /></a>
</div>

<a href="https://www.codecogs.com/eqnedit.php?latex=p=sigmoid(x)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?p=sigmoid(x)" title="p=sigmoid(x)" /></a>，其中x为模型的直接输出，那么我们对x求导可得：

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\frac&space;{&space;\partial&space;L&space;_&space;{&space;C&space;E&space;}&space;}&space;{&space;\partial&space;x&space;}&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;p&space;-&space;1&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\frac&space;{&space;\partial&space;L&space;_&space;{&space;C&space;E&space;}&space;}&space;{&space;\partial&space;x&space;}&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;p&space;-&space;1&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." title="\frac { \partial L _ { C E } } { \partial x } = \left\{ \begin{array} { l l } { p - 1 } & { \text { if } p ^ { * } = 1 } \\ { p } & { \text { if } p ^ { * } = 0 } \end{array} \right." /></a>
</div>

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex==p-p&space;^&space;{&space;*&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?=p-p&space;^&space;{&space;*&space;}" title="=p-p ^ { * }" /></a>
</div>

因此，我们定义g(gradient norm)如下所示：

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=g&space;=&space;\left|&space;p&space;-&space;p&space;^&space;{&space;*&space;}&space;\right|&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;1&space;-&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?g&space;=&space;\left|&space;p&space;-&space;p&space;^&space;{&space;*&space;}&space;\right|&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;1&space;-&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;1&space;}&space;\\&space;{&space;p&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;p&space;^&space;{&space;*&space;}&space;=&space;0&space;}&space;\end{array}&space;\right." title="g = \left| p - p ^ { * } \right| = \left\{ \begin{array} { l l } { 1 - p } & { \text { if } p ^ { * } = 1 } \\ { p } & { \text { if } p ^ { * } = 0 } \end{array} \right." /></a>
</div>

当g趋向于1时，代表样本较为困难，模型难以预测及（number(false positive) + number(true negtive)）,当g趋向于0时，代表样本较为简单，模型预测较为容易。如下图所示（GHM图1），数据集中（我认为应该是coco）图片困难度同样存在一个样本不平衡问题：

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure1.png" />
</div>

two-stage detector 比one-stage效果好的最大原因是RPN的存在，这样可以极大程度的减少正负样本的不平衡问题。因此我们减少难易样本的不平衡问题变得至关重要，我们简要阐述三篇文章的动机并进行分析。

**2.Online hard example mining(OHEM)**

OHEM（online hard example miniing）算法的核心思想是根据输入样本的损失进行筛选，筛选出hard example，表示对分类和检测影响较大的样本，然后将筛选得到的这些样本应用在随机梯度下降中训练。在实际操作中是将原来的一个ROI Network扩充为两个ROI Network，这两个ROI Network共享参数。其中前面一个ROI Network只有前向操作，主要用于计算损失；后面一个ROI Network包括前向和后向操作，以hard example作为输入，计算损失并回传梯度。网络结构图如下所示：

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure2.png" width="800" />
</div>

算法优点：
	1. 对于数据的类别不平衡问题不需要采用设置正负样本比例的方式来解决，这种在线选择方式针对性更强。
	2. 随着数据集的增大，算法的提升更加明显（作者是通过在COCO数据集上做实验和VOC数据集做对比，因为前者的数据集更大，而且提升更明显，所以有这个结论）。
算法缺点：
	1. 网络结构较为复杂，前向推断耗用内存空间。
	2. OHEM虽然增加了错分类的权重，但是OHEM算法忽略了容易分类的样本。

**3.focal loss**

在传统的样本不平衡问题上，我们常常采用重采样，或者针对loss函数进行改进。在目标检测领域，针对类别不平衡问题，作者提出了一种新的损失函数，这个损失函数是在标准交叉熵损失基础上修改得到的。通过减少易分类样本的权重，使得模型在训练时更专注于难分类的样本。想法很简单，具体可以参考我师兄张海鹏之前写过的一篇[博客](https://zhpmatrix.github.io/2017/08/13/focal-loss/)

**4.GHM(gradient harmonizing mechanism)**

下面来着重介绍一下今年AAAI2019的由港中文提出的GHM工作。还是给我很多的启发。

承上，工作还是针对困难样本，与focal loss相同的是，也是针对loss函数进行改进。

4.1 针对权重进行分析

首先定义梯度密度函数（Gradient density function）

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=G&space;D&space;(&space;g&space;)&space;=&space;\frac&space;{&space;1&space;}&space;{&space;l&space;_&space;{&space;\epsilon&space;}&space;(&space;g&space;)&space;}&space;\sum&space;_&space;{&space;k&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\delta&space;_&space;{&space;\epsilon&space;}&space;\left(&space;g&space;_&space;{&space;k&space;}&space;,&space;g&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?G&space;D&space;(&space;g&space;)&space;=&space;\frac&space;{&space;1&space;}&space;{&space;l&space;_&space;{&space;\epsilon&space;}&space;(&space;g&space;)&space;}&space;\sum&space;_&space;{&space;k&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\delta&space;_&space;{&space;\epsilon&space;}&space;\left(&space;g&space;_&space;{&space;k&space;}&space;,&space;g&space;\right)" title="G D ( g ) = \frac { 1 } { l _ { \epsilon } ( g ) } \sum _ { k = 1 } ^ { N } \delta _ { \epsilon } \left( g _ { k } , g \right)" /></a>
</div>

其中<a href="https://www.codecogs.com/eqnedit.php?latex=g_{k}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?g_{k}" title="g_{k}" /></a>表示第k个样本的梯度，而且

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\delta&space;_&space;{&space;\epsilon&space;}&space;(&space;x&space;,&space;y&space;)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;r&space;}&space;{&space;1&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;y&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;<&space;=&space;x&space;<&space;y&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;}&space;\\&space;{&space;0&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\delta&space;_&space;{&space;\epsilon&space;}&space;(&space;x&space;,&space;y&space;)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;r&space;}&space;{&space;1&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;y&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;<&space;=&space;x&space;<&space;y&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;}&space;\\&space;{&space;0&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." title="\delta _ { \epsilon } ( x , y ) = \left\{ \begin{array} { l r } { 1 } & { \text { if } y - \frac { \epsilon } { 2 } < = x < y + \frac { \epsilon } { 2 } } \\ { 0 } & { \text { otherwise } } \end{array} \right." /></a>
</div>

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=l&space;_&space;{&space;\epsilon&space;}&space;(&space;g&space;)&space;=&space;\min&space;\left(&space;g&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;1&space;\right)&space;-&space;\max&space;\left(&space;g&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;0&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?l&space;_&space;{&space;\epsilon&space;}&space;(&space;g&space;)&space;=&space;\min&space;\left(&space;g&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;1&space;\right)&space;-&space;\max&space;\left(&space;g&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;0&space;\right)" title="l _ { \epsilon } ( g ) = \min \left( g + \frac { \epsilon } { 2 } , 1 \right) - \max \left( g - \frac { \epsilon } { 2 } , 0 \right)" /></a>
</div>

所以梯度密度函数GD(g)就是表示梯度落在区域<a href="https://www.codecogs.com/eqnedit.php?latex=\left[&space;g&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;g&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\left[&space;g&space;-&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;,&space;g&space;&plus;&space;\frac&space;{&space;\epsilon&space;}&space;{&space;2&space;}&space;\right)" title="\left[ g - \frac { \epsilon } { 2 } , g + \frac { \epsilon } { 2 } \right)" /></a>的样本数量。再定义梯度密度协调参数（gradient densitu harmonizing parameter）<a href="https://www.codecogs.com/eqnedit.php?latex=\beta" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\beta" title="\beta" /></a>

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\beta&space;_{i}=\frac{N}{GD(g_{i})}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\beta&space;_{i}=\frac{N}{GD(g_{i})}" title="\beta _{i}=\frac{N}{GD(g_{i})}" /></a>
</div>

其中N表示总的样本数，表示标准化。可以看出，梯度密度大的样本的权重会被降低，密度小的样本的群众会增加，以此将GHM的思想对目标检测的分类和回归loss做改进行程GHM-C 和GHM-R.

因此，我们在训练样本时需要提前得到所有样本的梯度密度分布。而计算所有样本的梯度密度分布的时间复杂度为O(N^2).而通过先排序再统计梯度的方式，最佳的排序算法实际复杂度为O(NlogN),统计时间复杂度为O(N)，但是这种方法并行收益不大，因此本篇论文通过简化求出近似的梯度密度分布来加快计算。这种方法的原理也比较简单，就是把X轴划分为M个区域，对于落在每个区域样本的权重采取相同的修正方式，类似于直方图。具体推导公式如下所示。X轴的梯度分为M个区域，每个区域长度即为<a href="https://www.codecogs.com/eqnedit.php?latex=\varepsilon" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\varepsilon" title="\varepsilon" /></a>，第j个区域范围即为<a href="https://www.codecogs.com/eqnedit.php?latex=r_{j}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?r_{j}" title="r_{j}" /></a>，用<a href="https://www.codecogs.com/eqnedit.php?latex=R_{j}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?R_{j}" title="R_{j}" /></a>表示落在第j个区域内的样本数量。定义ind(g)表示梯度为g的样本所落区域的序号，那么即可得出新的参数<a href="https://www.codecogs.com/eqnedit.php?latex=\beta_{i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\beta_{i}" title="\beta_{i}" /></a>新的计算方法的时间复杂度为 O(MN) 。

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure3.png" />
</div>

另外，当batch size足够大时，batch样本的梯度密度可以作为整个数据集上的梯度密度来进行替代使用，使得模型能够直接训练，不需要繁琐步骤。而基于batch直接计算出来的梯度密度可能不稳定，所以采用滑动平均的方式处理梯度计算。

4.2 GHM-C

把GHM应用于分类的loss上即为GHM-C，定义如下所示：

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=L&space;_&space;{&space;G&space;H&space;M&space;-&space;C&space;}&space;=&space;\frac&space;{&space;1&space;}&space;{&space;N&space;}&space;\sum&space;_&space;{&space;i&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\beta&space;_&space;{&space;i&space;}&space;L&space;_&space;{&space;C&space;E&space;}&space;\left(&space;p&space;_&space;{&space;i&space;}&space;,&space;p&space;_&space;{&space;i&space;}&space;^&space;{&space;*&space;}&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L&space;_&space;{&space;G&space;H&space;M&space;-&space;C&space;}&space;=&space;\frac&space;{&space;1&space;}&space;{&space;N&space;}&space;\sum&space;_&space;{&space;i&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\beta&space;_&space;{&space;i&space;}&space;L&space;_&space;{&space;C&space;E&space;}&space;\left(&space;p&space;_&space;{&space;i&space;}&space;,&space;p&space;_&space;{&space;i&space;}&space;^&space;{&space;*&space;}&space;\right)" title="L _ { G H M - C } = \frac { 1 } { N } \sum _ { i = 1 } ^ { N } \beta _ { i } L _ { C E } \left( p _ { i } , p _ { i } ^ { * } \right)" /></a>
</div>

根据GHM-C的loss计算方式，候选样本中的简单负样本和非常困难的异常样本的权重都会被降低，即loss会被降低，对于模型训练的影响也会被大大减小。正常困难样本的权重得到提升，这样模型就会更加专注于那些更为有效的正常困难样本，以提升模型的性能。GHM-C loss对模型梯度的修正效果如下图所示，横轴表示原始的梯度loss，纵轴表示修正后的。由于样本的极度不均衡，这篇论文中所有的图纵坐标都是取对数画的图。

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure4.png" />
</div>

4.2 GHM-R

GHM的思想同样适用于anchor的坐标回归。坐标回归loss常用smooth_l1,如下所示:

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=L&space;_&space;{&space;r&space;e&space;g&space;}&space;=&space;\sum&space;_&space;{&space;i&space;\in&space;\{&space;x&space;,&space;y&space;,&space;w&space;,&space;h&space;\}&space;}&space;S&space;L&space;_&space;{&space;1&space;}&space;\left(&space;t&space;_&space;{&space;i&space;}&space;-&space;t&space;_&space;{&space;i&space;}&space;^&space;{&space;*&space;}&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L&space;_&space;{&space;r&space;e&space;g&space;}&space;=&space;\sum&space;_&space;{&space;i&space;\in&space;\{&space;x&space;,&space;y&space;,&space;w&space;,&space;h&space;\}&space;}&space;S&space;L&space;_&space;{&space;1&space;}&space;\left(&space;t&space;_&space;{&space;i&space;}&space;-&space;t&space;_&space;{&space;i&space;}&space;^&space;{&space;*&space;}&space;\right)" title="L _ { r e g } = \sum _ { i \in \{ x , y , w , h \} } S L _ { 1 } \left( t _ { i } - t _ { i } ^ { * } \right)" /></a>
</div>

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=S&space;L&space;_&space;{&space;1&space;}&space;(&space;d&space;)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;\frac&space;{&space;d&space;^&space;{&space;2&space;}&space;}&space;{&space;2&space;\delta&space;}&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;|&space;d&space;|&space;<&space;=&space;\delta&space;}&space;\\&space;{&space;|&space;d&space;|&space;-&space;\frac&space;{&space;\delta&space;}&space;{&space;2&space;}&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?S&space;L&space;_&space;{&space;1&space;}&space;(&space;d&space;)&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;\frac&space;{&space;d&space;^&space;{&space;2&space;}&space;}&space;{&space;2&space;\delta&space;}&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;|&space;d&space;|&space;<&space;=&space;\delta&space;}&space;\\&space;{&space;|&space;d&space;|&space;-&space;\frac&space;{&space;\delta&space;}&space;{&space;2&space;}&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." title="S L _ { 1 } ( d ) = \left\{ \begin{array} { l l } { \frac { d ^ { 2 } } { 2 \delta } } & { \text { if } | d | < = \delta } \\ { | d | - \frac { \delta } { 2 } } & { \text { otherwise } } \end{array} \right." /></a>
</div>

其中<a href="https://www.codecogs.com/eqnedit.php?latex=t_{i}&space;=&space;(t_{x},&space;t_{y},&space;t_{w},&space;t_{h})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?t_{i}&space;=&space;(t_{x},&space;t_{y},&space;t_{w},&space;t_{h})" title="t_{i} = (t_{x}, t_{y}, t_{w}, t_{h})" /></a>表示模型预测坐标偏移量, <a href="https://www.codecogs.com/eqnedit.php?latex=t_{i}&space;=&space;(t_{x}^{*},&space;t_{y}^{*},&space;t_{w}^{*},&space;t_{h}^{*})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?t_{i}&space;=&space;(t_{x}^{*},&space;t_{y}^{*},&space;t_{w}^{*},&space;t_{h}^{*})" title="t_{i} = (t_{x}^{*}, t_{y}^{*}, t_{w}^{*}, t_{h}^{*})" /></a> 表示anchor实际坐标偏移量， <a href="https://www.codecogs.com/eqnedit.php?latex=\delta" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\delta" title="\delta" /></a>表示 SL_1 的函数分界点，常取1/9。定义<a href="https://www.codecogs.com/eqnedit.php?latex=d=t_{i}-t_{i}^{*}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?d=t_{i}-t_{i}^{*}" title="d=t_{i}-t_{i}^{*}" /></a>，则<a href="https://www.codecogs.com/eqnedit.php?latex=SL_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?SL_{1}" title="SL_{1}" /></a>的梯度求导为:

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\frac&space;{&space;\partial&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;t&space;_&space;{&space;i&space;}&space;}&space;=&space;\frac&space;{&space;\partial&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;d&space;}&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;\frac&space;{&space;d&space;}&space;{&space;\delta&space;}&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;|&space;d&space;|&space;<&space;=&space;\delta&space;}&space;\\&space;{&space;\operatorname&space;{&space;sgn&space;}&space;(&space;d&space;)&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\frac&space;{&space;\partial&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;t&space;_&space;{&space;i&space;}&space;}&space;=&space;\frac&space;{&space;\partial&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;d&space;}&space;=&space;\left\{&space;\begin{array}&space;{&space;l&space;l&space;}&space;{&space;\frac&space;{&space;d&space;}&space;{&space;\delta&space;}&space;}&space;&&space;{&space;\text&space;{&space;if&space;}&space;|&space;d&space;|&space;<&space;=&space;\delta&space;}&space;\\&space;{&space;\operatorname&space;{&space;sgn&space;}&space;(&space;d&space;)&space;}&space;&&space;{&space;\text&space;{&space;otherwise&space;}&space;}&space;\end{array}&space;\right." title="\frac { \partial S L _ { 1 } } { \partial t _ { i } } = \frac { \partial S L _ { 1 } } { \partial d } = \left\{ \begin{array} { l l } { \frac { d } { \delta } } & { \text { if } | d | < = \delta } \\ { \operatorname { sgn } ( d ) } & { \text { otherwise } } \end{array} \right." /></a>
</div>

其中sgn表示符号函数。可以看出在其所在区域内，对于的所有样本梯度绝对值都为1，这使我们无法通过梯度来区分样本，同时d理论上可以到无穷大，这也使我们不能使用上面提到的RU。所以论文对则<a href="https://www.codecogs.com/eqnedit.php?latex=SL_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?SL_{1}" title="SL_{1}" /></a>进行变形，计算方法及梯度求导如下所示:

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=A&space;S&space;L&space;_&space;{&space;1&space;}&space;(&space;d&space;)&space;=&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;-&space;\mu" target="_blank"><img src="https://latex.codecogs.com/gif.latex?A&space;S&space;L&space;_&space;{&space;1&space;}&space;(&space;d&space;)&space;=&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;-&space;\mu" title="A S L _ { 1 } ( d ) = \sqrt { d ^ { 2 } + \mu ^ { 2 } } - \mu" /></a>
</div>

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\frac&space;{&space;\partial&space;A&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;d&space;}&space;=&space;\frac&space;{&space;d&space;}&space;{&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\frac&space;{&space;\partial&space;A&space;S&space;L&space;_&space;{&space;1&space;}&space;}&space;{&space;\partial&space;d&space;}&space;=&space;\frac&space;{&space;d&space;}&space;{&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;}" title="\frac { \partial A S L _ { 1 } } { \partial d } = \frac { d } { \sqrt { d ^ { 2 } + \mu ^ { 2 } } }" /></a>
</div>

<a href="https://www.codecogs.com/eqnedit.php?latex=ASL_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?ASL_{1}" title="ASL_{1}" /></a>与<a href="https://www.codecogs.com/eqnedit.php?latex=SL_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?SL_{1}" title="SL_{1}" /></a>的性质很相似，当d较大时都近似为L1 loss，d较小是都近似为L2 loss，而且<a href="https://www.codecogs.com/eqnedit.php?latex=ASL_{1}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?ASL_{1}" title="ASL_{1}" /></a>的范围在[0,1)，适合采用RU方法，在实际使用中，采用μ=0.02。定义梯度的绝对值gr为:

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=g&space;r&space;=&space;\left|&space;\frac&space;{&space;d&space;}&space;{&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;}&space;\right|" target="_blank"><img src="https://latex.codecogs.com/gif.latex?g&space;r&space;=&space;\left|&space;\frac&space;{&space;d&space;}&space;{&space;\sqrt&space;{&space;d&space;^&space;{&space;2&space;}&space;&plus;&space;\mu&space;^&space;{&space;2&space;}&space;}&space;}&space;\right|" title="g r = \left| \frac { d } { \sqrt { d ^ { 2 } + \mu ^ { 2 } } } \right|" /></a>
</div>

所以使用GHM的思想来修正loss函数，可以得到:

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=L&space;_&space;{&space;G&space;H&space;M&space;-&space;R&space;}&space;=&space;\frac&space;{&space;1&space;}&space;{&space;N&space;}&space;\sum&space;_&space;{&space;i&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\beta&space;_&space;{&space;i&space;}&space;A&space;S&space;L&space;_&space;{&space;1&space;}&space;\left(&space;d&space;_&space;{&space;i&space;}&space;\right)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L&space;_&space;{&space;G&space;H&space;M&space;-&space;R&space;}&space;=&space;\frac&space;{&space;1&space;}&space;{&space;N&space;}&space;\sum&space;_&space;{&space;i&space;=&space;1&space;}&space;^&space;{&space;N&space;}&space;\beta&space;_&space;{&space;i&space;}&space;A&space;S&space;L&space;_&space;{&space;1&space;}&space;\left(&space;d&space;_&space;{&space;i&space;}&space;\right)" title="L _ { G H M - R } = \frac { 1 } { N } \sum _ { i = 1 } ^ { N } \beta _ { i } A S L _ { 1 } \left( d _ { i } \right)" /></a>
</div>

如下图所示，GHM-R loss加大了简单样本和正常困难样本的权重，大大降低了异常样本的权重，使模型的训练更加合理.

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure5.png" />
</div>

实验结果如下图所示：

<div style="color:#0000FF" align="center">
<img src="/image/2018-12-22/figure6.png" width="800" />
</div>

**5.总结**

之前师兄提起过focal loss在工业界应用不广泛，我觉得主要原因可能还是由于工业界所使用的数据集较为单一导致的（并没有过多的困难样本？？）。针对目标检测领域不平衡样本的主要工作总结就是这些，主要工作点还是倾向于对难易样本问题的解决。同样，在分割领域，能否针对困难像素做一些类似工作？？

**参考文献**

1.[Training Region-based Object Detectors with Online Hard Example Mining](https://arxiv.org/abs/1604.03540)

2.[Focal Loss for Dense Object Detection](https://arxiv.org/abs/1708.02002)

3.[Gradient Harmonized Single-stage Detector](https://arxiv.org/abs/1811.05181)