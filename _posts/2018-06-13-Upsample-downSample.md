---
layout: post
title: "[DL]关于上采样及下采样过程关键点总结"
excerpt: "围绕卷积神经网中的采样过程进行简要分析，同时提了一些想法。"
date: 2018-06-13 14:39:00
mathjax: true
---

关于上采样以及下采样不同方式的效果图，大家可以参考这篇[博客](https://github.com/vdumoulin/conv_arithmetic),我就不一一阐述了。


**上采样(upsampling)**

我们可以采用很多不一样的方法对特征图的分辨率进行上采样


1.Uppooling:

池化操作通过汇总局部区域的单个值（平均值或者最大值）下采样分辨率，而上池化操作通过将单个值分配给更高的分辨率对分辨率进行上采样。

<div style="color:#0000FF" align="center">
<img src="/image/2018-06-13/unpooling.jpg" width="750"/> 
</div>

2.Transpose conv

转置卷积是最常用的方法，转置卷积又称为反卷积或上卷积，但他并不是卷积操作的逆过程。由于转置卷积允许我们开发学习过的上采样。卷积运算会将卷积核权重与当前值进行点积，并为相应输出位置产生单个值，转置卷积会先从低分辨率的特征映射中得到某个值，再用该值与卷积核中所有权重相乘，然后将这些加权值映射到输出特征图中。对在输出特征映射图中产生重叠的卷积核尺寸而言，重叠值是简单的叠加。不幸的是，这会在输出中产生棋盘效应，所以最好保证卷积核不会产生重叠.通常采用对低分辨率图片进行padding的方式，如下图：

<div style="color:#0000FF" align="center">
<img src="/image/2018-06-13/padding_strides_transposed.gif"/> 
</div>

3.Bilinear Upsampling

双线性插值在上采样的过程当中也比较常用，但他独立于卷积运算之外，具体操作过程请参考这篇[博客](http://www.cnblogs.com/yssongest/p/5303151.html)。因其采用浮点数运算，所以这种方式对内存消耗较大。由于双线性插值也会与卷积运算结合，据说双线性插值是转置卷积所耗费时间的4倍，有待考证~


4.还有一些针对转置卷积运算的一些改进操作，例如Decomposed Transposed Conv、 Separable Transposed Convolution、 Depth To Space 等方法，在参数数量上进行优化。从模型压缩的角度，对卷积运算进行改进。


**下采样(downsampling)**

1.Dilated Conv:

对特征映射进行下采样的一个好处是在给定常量卷积核尺寸的情况下扩展了感受野（对于输入）。由于大尺寸卷积核的参数效率较低，所以这种方法比增加卷积核尺寸更加合理。然而，这种扩展的代价是降低了空间分别率。扩张卷积提供了另外一种在保留完整空间维度的同时还能获得广泛视野的方法。如下图，扩张卷积根据指定的扩张率（dilateion rate）用值将空间间隔开。

<div style="color:#0000FF" align="center">
<img src="/image/2018-06-13/dilation.gif"/> 
</div>

**想法及总结**

针对模型采样，我觉得还是有很多工作可以去做的，毕竟是炼丹的基础，比如说最近的Deformable Convolutional Network，他能不能说是SIFT的一种借鉴？？另外昨天跟师兄讨论了一波关于在目标检测中先resize和不进行resize之间精度对比，并没有得出好的结论。等我有时间把这个工作推上来再进行吹逼吧。OK,写到这，撤了~~~























