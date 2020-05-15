---
layout: post
title: "[笔记]聊一聊神经网络中的Normalization"
excerpt: "对于神经网络中的BN、IN、LN、GN、Inplace-ABN五种Normalization算法总结"
date: 2020-05-14 16:57:00
mathjax: true
---

**1. 前言**

随着训练的进行，网络中的参数也随着梯度下降在不停更新。一方面，当底层网络中的参数发生微弱变化时，由于每一层中的线性变换与非线性激活函数，这些微小的变化随着网络随着网络层数的加深而被放大。另一方面，参数的变化导致每一层的输入分布会发生改变，进而上层的网络需要不停地去适应这些分布变化，使得我们的模型训练变得困难，这一现象叫做（Internal Convariate Shift）。

Internal Convariate Shift所带来的问题：

a. 上层网络需要不停调整来适应输入数据分布的变化，导致网络学习速率的降低。

b. 网络的训练过程容易陷入梯度饱和区，减缓网络收敛速度。

而在机器学习里面，对输入数据进行变换的常用手段是白化操作（即通过归一化方式，使得输入特征分布具有相同的均值与方差，去除掉特征的相关性），通过白化操作，我们可以减缓Internal Convariate Shift,加速网络收敛。但是白化操作计算成本太高，另外就是白化过程会改变网络的特征分布，改变网络本身的表达能力。因此，基于上述原因，改进版的白化操作Batch Normalization变应用而生。

**2. Batch Normalization**

对于给定的feature map(N,C,H,W)，其中N代表batch的大小，C代表特征的通道数，H,W分别为特征矩阵的长、宽。

在通道维度上的均值为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;_{c}\left&space;(&space;x&space;\right&space;)=\frac{1}{NWH}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;_{c}\left&space;(&space;x&space;\right&space;)=\frac{1}{NWH}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" title="\mu _{c}\left ( x \right )=\frac{1}{NWH}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" /></a>
</div>
在通道维度上的方差为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma&space;_{c}\left&space;(&space;x&space;\right&space;)&space;=&space;\sqrt{\frac{1}{NHW}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}\left&space;(x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;\right&space;)^{2}&plus;\varepsilon&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma&space;_{c}\left&space;(&space;x&space;\right&space;)&space;=&space;\sqrt{\frac{1}{NHW}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}\left&space;(x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;\right&space;)^{2}&plus;\varepsilon&space;}" title="\sigma _{c}\left ( x \right ) = \sqrt{\frac{1}{NHW}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}\left (x_{nchw}-\mu _{c}\left ( x \right ) \right )^{2}+\varepsilon }" /></a>
</div>

由于归一化操作会减弱网络每一层输入数据表达能力，因此，BN在均一化的基础上增加线性变换操作，让这些数据能够尽可能恢复本身的表达能力，因此，最终的变换公式为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\tilde{x}_{nchw}&space;=&space;\gamma&space;\frac{x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;}{\sqrt{\sigma&space;_{c}(x)^{2}&plus;\varepsilon&space;}}&plus;\beta" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\tilde{x}_{nchw}&space;=&space;\gamma&space;\frac{x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;}{\sqrt{\sigma&space;_{c}(x)^{2}&plus;\varepsilon&space;}}&plus;\beta" title="\tilde{x}_{nchw} = \gamma \frac{x_{nchw}-\mu _{c}\left ( x \right ) }{\sqrt{\sigma _{c}(x)^{2}+\varepsilon }}+\beta" /></a>
</div>
因此，BN又引入了两个可学习的参数<a href="https://www.codecogs.com/eqnedit.php?latex=\gamma" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\gamma" title="\gamma" /></a>，<a href="https://www.codecogs.com/eqnedit.php?latex=\beta" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\beta" title="\beta" /></a>。在模型训练结束后，我们保留每组mini-batch训练数据在网络中每一层的<a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;_{batch}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;_{batch}" title="\mu _{batch}" /></a>以及<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma&space;^{2}_{batch}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma&space;^{2}_{batch}" title="\sigma ^{2}_{batch}" /></a>，因此，在模型推断时，我们可以使用整个训练样本的统计量对测试机数据进行归一化，具体来说使用均值与方差的无偏估计：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\mu_{test}=E(\mu_{batch})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu_{test}=E(\mu_{batch})" title="\mu_{test}=E(\mu_{batch})" /></a>
</div>
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma&space;^{2}_{test}=\frac{m}{m-1}E(\sigma&space;^{2}_{batch})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma&space;^{2}_{test}=\frac{m}{m-1}E(\sigma&space;^{2}_{batch})" title="\sigma ^{2}_{test}=\frac{m}{m-1}E(\sigma ^{2}_{batch})" /></a>
</div>

****吹一吹BN的作用***

1. BN使得网络中每层输入数据的分布相对稳定，加速模型学习速度。
2. BN使得模型对网络中的参数不那么敏感，简化调参过程，使得网络学习更加稳定。
3. BN的使用，可以让我们在模型设计时，使用Sigmoid、tanh等饱和性激活函数，能够有效的避免梯度消失问题。
4. BN具有一定层度的正则化（归一化操作在一定程度上减少了随机噪音）。

**3. Layer Normalization**

BN的一个缺点就是需要较大的batch size才能合理的评估训练数据的方差和均值，这导致机器的显存可能不够用，因此，很难应用于训练数据长度不同的RNN序列上，针对这个问题，Layer Normalization在batch维度进行均值、方差的计算：

在batch维度上的均值为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;_{n}\left&space;(&space;x&space;\right&space;)=\frac{1}{NWH}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;_{n}\left&space;(&space;x&space;\right&space;)=\frac{1}{NWH}\sum_{c=1}^{C}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" title="\mu _{c}\left ( x \right )=\frac{1}{NWH}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" /></a>
</div>
在batch维度上的方差为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma&space;_{c}\left&space;(&space;x&space;\right&space;)&space;=&space;\sqrt{\frac{1}{NHW}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}\left&space;(x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;\right&space;)^{2}&plus;\varepsilon&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma&space;_{n}\left&space;(&space;x&space;\right&space;)&space;=&space;\sqrt{\frac{1}{NHW}\sum_{c=1}^{C}\sum_{h=1}^{H}\sum_{w=1}^{W}\left&space;(x_{nchw}-\mu&space;_{c}\left&space;(&space;x&space;\right&space;)&space;\right&space;)^{2}&plus;\varepsilon&space;}" title="\sigma _{c}\left ( x \right ) = \sqrt{\frac{1}{NHW}\sum_{n=1}^{N}\sum_{h=1}^{H}\sum_{w=1}^{W}\left (x_{nchw}-\mu _{c}\left ( x \right ) \right )^{2}+\varepsilon }" /></a>
</div>

其余计算方式与BN相同。值得一提的是：Nvidia在apex中提供了Layer Normalization的实现方法：

```
class apex.normalization.FusedLayerNorm(normalized_shape, eps=1e-05, elementwise_affine=True)[source]
```

**4. Instance Normalization**

Instance Normalization最初应用于风格迁移中，作者发现，在生成模型中，feature map的各个通道件的均值和方差会影响最终图像生成任务的风格：

因此，IN保留batch、以及通道维度，仅在H、W维度上计算均值：

<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\mu&space;_{nc}(x)&space;=&space;\frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu&space;_{nc}(x)&space;=&space;\frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" title="\mu _{nc}(x) = \frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" /></a>
</div>
方差为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma_{nc}(x)&space;=\sqrt{\frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu&space;_{nc}(x))^{2}&plus;\varepsilon&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma_{nc}(x)&space;=\sqrt{\frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu&space;_{nc}(x))^{2}&plus;\varepsilon&space;}" title="\sigma_{nc}(x) =\sqrt{\frac{1}{HW}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu _{nc}(x))^{2}+\varepsilon }" /></a>
</div>

**5. Group Normalization**

GN作为LN的特例，在通道上进行分组，以减少LN的计算量：
均值为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\mu_{ng}(x)&space;=\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g&plus;1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\mu_{ng}(x)&space;=\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g&plus;1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" title="\mu_{ng}(x) =\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g+1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}x_{nchw}" /></a>
</div>
方差为：
<div style="color:#0000FF" align="center">
<a href="https://www.codecogs.com/eqnedit.php?latex=\sigma_{ng}(x)&space;=\sqrt{\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g&plus;1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu&space;_{ng}(x))^{2}&plus;\varepsilon&space;}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma_{ng}(x)&space;=\sqrt{\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g&plus;1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu&space;_{ng}(x))^{2}&plus;\varepsilon&space;}" title="\sigma_{ng}(x) =\sqrt{\frac{1}{(C/G)HW}\sum_{c=gC/G}^{(g+1)C/G}\sum_{h=1}^{H}\sum_{w=1}^{W}(x_{nchw}-\mu _{ng}(x))^{2}+\varepsilon }" /></a>
</div>

**6.IN-place ABN**

在先进的网络中，大多数神经网络重复使用BN+激活层组合，而现有的深度学习框架对此的内存优化策略不佳，因此，采用In-aplace ABN层代替BN+激活层，通过存储少量的计算结果（丢弃部分中间结果，在反向传播时倒置计算恢复需要的参数），节省存储空间，少量的增加计算量（以时间换空间的策略）。具体内容讲解见[网络优化-- (INPLACE-ABN)In-Place Activated BatchNorm for Memory-Optimized Training of DNNs](https://blog.csdn.net/u011974639/article/details/79545363)。

**7. 参考文献**

1. [Layer Normalization](https://arxiv.org/abs/1607.06450)

2. [Group Normalization](https://arxiv.org/abs/1803.08494)

3. [Instance Normalization: The Missing Ingredient for Fast Stylization](https://arxiv.org/abs/1607.08022)

4. [In-Place Activated BatchNorm for Memory-Optimized Training of DNNs](https://arxiv.org/pdf/1712.02616.pdf)

5. [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167)
