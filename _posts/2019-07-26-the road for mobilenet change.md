---
layout: post
title: "[Summary]MobileNet 系列网络的演化之路"
excerpt: "本文详细总结了MobileNet V1 到 MobileNet V3 的改进点，增加最新的GhostNet、MobileNeXt"
date: 2019-07-26 15:34:00
mathjax: true
---

***1. Abstract***

自从谷歌公司在2017年提出MobileNet之后， MobileNet可谓是轻量级网络中的Inception,经历了一代又一代的更新，成为学习轻量级神经网络的必经之路。本文详细总结了MobileNet V1 到 MobileNet V3 的contribution，内容如下：

***2. [MobileNet v1](https://arxiv.org/abs/1704.04861)***

V1的主要贡献点就是提出了现阶段大多数轻量级神经网络中采用的可分离卷积。可以说V1是现阶段轻量级神经网络的基石，当前，MobileNet V1的引用量已经达到1940次，可见MobileNet的影响之大。通常情况下，可分离卷积可以分为空间可分离卷积和深度可分离卷积：

【空间可分离卷积】顾名思义就是将一个大的卷积核变成两个小的卷积核，比如将一个3x3的核分成一个3x1和一个1x3的核。

【深度可分离卷积】就是将普通卷积拆分成一个深度卷积（depthwise）和一个逐点卷积(pointwise)。

深度卷积：与标准卷积网络不同，我们将卷积核拆分成单通道的形式，在不改变输入特征图像的深度的情况下，对每一个通道进行卷积操作，这样就得到了和输入特征图一致的输出特征图。（ps：深度卷积能够有效的减少参数量，但也同时带来了通道特征之间的不交互的问题，即使后面采用pointwise卷积，也没有很好的提升网络的性能。 因此，shufflenet针对深度卷积所带来的性能衰减又继续做了随机打乱特征的改进）

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/depthwise.jpg"/>
</div>

逐点卷积：就是1x1卷积。主要作用就是对特征图进行升维和降维。

因此，MobileNet V1 将标准卷积做了如下替换,此处的ReLU6为ReLu的改进，作者认为ReLu6在低精度计算下具有更强的鲁棒性。其中：<a href="https://www.codecogs.com/eqnedit.php?latex=ReLu6(x)&space;=&space;min(max(0,x),6)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?ReLu6(x)&space;=&space;min(max(0,x),6)" title="ReLu6(x) = min(max(0,x),6)" /></a>：

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/unit.jpg"/>
</div>

根据实验可以看到，在imagenet分类任务上，深度可分离卷积和标准卷积相比，参数和计算量能下降为后者的九分之一到八分之一左右，但是准确率只有下降极小的1%。

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/v1 experiments.jpg"/>
</div>

v1的网络结构如下图所示，首先是一个标准的3x3卷积，同时步长为2进行下采样。然后就是堆积深度可分离卷积，并且其中的部分深度卷积会采用步长为2继续进行下采样。最后采用平均池化层将feature变成1x1,后面全连接等操作。整个网络有28层，其中深度可分离卷积13层。

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/v1 experiments.jpg"/>
</div>

作者在论文中采用了大量的实验，在此就不赘述了，感兴趣的同学可以看下原论文。

***3. [MobileNet v2](https://arxiv.org/abs/1801.04381)***

1. v1提出的深度可分离卷积在使用较少的参数量、计算量，提高网络运行速度的同时，又能得到一个接近于标准卷积的结果，这看起来很美好。但是在使用过程中发现，深度可分离卷积的部分卷积核在训练完成之后出现卷积核为空的情况：作者认为这是由于ReLU造成的，并由此得出结论，ReLU在低维特征中具有保存完整信息的能力（具体分析其实我也没怎么看懂，比较抽象！）。因此，作者将V1中最后的ReLU6换成Linear,作者将这个部分称为linear bottleneck。

2. 作者参考ResNet网络，加入Expansion Layer 与 residuals operation, 深度卷积本身没有改变通道的能力，来的是多少通道输出就是多少通道，如果通道很少的话，深度可分离卷积就只能在低维度特征上工作，而ReLU很有可能将这部分特征过滤掉。因此，我们要扩张通道，在进行深度可分离卷积之前，先将特征进行升维，在一个高维空间中进行特征提取。与ResNet不同的是，ResNet先降维（0.25倍），卷积，再升维；MobileNet V2则是先升维度（6倍），卷积，降维。刚好与ResNet相反，因此，作者将其命名为Inverted resuduals.

下图是V2论文中所提到的不同轻量级神经网络的部分组件。右上角的V1与右下角的V2相比，没有shortcut并且最后带有ReLU6;V2加入了1x1升维，引入shortcut并且去掉了最后的ReLU。步长为1时，先进行1x1卷积升维，再进行深度卷积提取特征，再通过Linear的逐点卷积降维，然后将input与output相加，形成残差结构。步长为2时，因为input与output的尺寸不符，因此不添加shortcut结构，其余均一致。

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/v2 architecture.jpg"/>
</div>

关于V2的网络结构本文就不详细列出。实验中，作者分别进行分类检测和分割任务验证V2的鲁棒性。值得一提的是，作者提出了SSDLite,针对SSD结构做了修改，将SSD的预测层中所有标准卷积替换成深度可分离卷积。作者说这样参数量和计算成本大大降低，计算更高效。基于MobileNet V2的SSDLite在COCO数据集上超过了YOLOv2,并且大小小10倍，速度快20倍。

***4. [MobileNet v3](https://arxiv.org/abs/1905.02244)***

MobileNet V3,是谷歌在2019年3月提出的网络结构。效果确实强的一比！但是，凭良心来讲，论文的创新点并不是很足，只是将之前工业界或者刷榜的各种trick加上，然后整合成V3，论文并没有很好的对V1,V2的弱点进行过多的分析。V3的相关技术总结如下：

1. 网络的结构基于NAS实现的[MnasNet](https://arxiv.org/abs/1807.11626)(效果比V2要好)。

2. 引入V1的深度可分离卷积。

3. 引入V2的具有线性瓶颈的倒残差结构。

4. 引入基于squeeze and excitation结构的轻量级注意力模型[SENet](https://arxiv.org/abs/1709.01507)。

5. 使用了一种新的激活函数h-swish(x)。

6. 网络结构搜索中，结合两种技术：资源受限的NAS（platform-aware NAS）与NetAdapt。

7. 修改了v1与v2网络的最后阶段。

【h-swish】 h-swish是基于swish的改进，swish是由谷歌在2017年的[Searching for Activation functions](https://arxiv.org/abs/1710.05941)所提出，其中，<a href="https://www.codecogs.com/eqnedit.php?latex=swish(x)=x*sigmoid\left&space;(&space;\beta&space;x&space;\right&space;)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?swish(x)=x*sigmoid\left&space;(&space;\beta&space;x&space;\right&space;)" title="swish(x)=x*sigmoid\left ( \beta x \right )" /></a>。论文的作者认为，Swish具备无上界有下界、平滑、非单调的特性。并且Swish在深层模型上的效果优于ReLU。仅仅使用Swish单元替换ReLU就能把MobileNet,NASNetA在 ImageNet上的top-1分类准确率提高0.9%，Inception-ResNet-v的分类准确率提高0.6%。V3也利用swish当作为ReLU的替代时，它可以显著提高神经网络的精度。但是呢，作者认为这种非线性激活函数虽然提高了精度，但在嵌入式环境中，是有不少的成本的。原因就是在移动设备上计算sigmoid函数是非常明智的选择。所以提出了h-swish:<a href="https://www.codecogs.com/eqnedit.php?latex=h-swish(x)=x\frac{ReLU6(x&plus;3))}{6}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?h-swish(x)=x\frac{ReLU6(x&plus;3))}{6}" title="h-swish(x)=x\frac{ReLU6(x+3))}{6}" /></a>。

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/h-swish.jpg"/>
</div>

论文指出，hard形式是soft形式的低精度化。swish的表现和其他非线性相比，能够将过滤器的数量减少到16个的同时保持与使用ReLU或swish的32个过滤器相同的精度，这节省了3毫秒的时间和1000万MAdds的计算量。并且同时，作者认为随着网络的深入，应用非线性激活函数的成本会降低，能够更好的减少参数量。作者发现swish的大多数好处都是通过在更深的层中使用它们实现的。因此，在V3的架构中，只在模型的后半部分使用h-swish(HS)。

【网络改进】作者认为，当前模型是基于V2模型中的倒残差结构。使用1x1卷积来构建最后层，这样可以便于拓展到更高维的特征空间，这样做的好处是在预测时，有更多丰富的特征来预测，但是同时也引入了额外的计算成本与延时。所以要保留高维特征的前提下减小延时。首先，还是将1×1层放在到最终平均池之后。这样的话最后一组特征现在不是7x7（下图V2结构红框），而是以1x1计算。（其实说白了就是在原来的基础上多加了个1x1卷积）。论文中，基于MobileNet v3, 作者分别提出large和small两个模型：

<div style="color:#0000FF" align="center">
<img src="/image/2019-07-26/v3 architecture.jpg", width="750" />
</div>

作者同样在分类检测和分割任务上检测V3的鲁棒性，由于代码没有开源，笔者采用复现代码，发现分类和检测任务都能达到甚至超出paper所展现的精度，但是在分割任务上，github上面开源的代码均没有达到paper中的精度，当然这个结果值得商榷。

***6. GhostNet***

[GhostNet](https://arxiv.org/abs/1911.11907)是华为在CVPR 2020上的工作，针对卷积神经网络严重的特征冗余，提出的一种轻量级的卷积神经网络，旨在利用廉价操作生成部分特征图，减少模型的计算复杂度。论文很好理解，可以看官方的[博客](https://zhuanlan.zhihu.com/p/109325275)，廉价的操作便是深度可分离卷积。目前代码已经开源，论文基本上延续着MobileNet系列的结构。直接看代码更好理解，在此我就不过多阐述。

***7. MobileNeXt***

[MobileNeXt](https://arxiv.org/abs/2007.02269)是颜水成团队近期公布的论文。说实话，论文并不是很打动我，创新型不是很足，而且印证的观点也不是很明确。工作抛弃了倒残差网络先升维后降维的思想，改成残差网络先降维再升维的常规操作，在降维之前，增加了一个深度可分离卷积，在升维之后呢，继续使用深度可分离卷积，然后在两个深度可分离卷积之间有shortcut，作者说，这样是在高维特征之间进行特征融合，减少特征传递的损耗...，然后就是采用Identity tensor multiplier的方式进行特征融合了，原谅我看了好长时间我也没看懂这是个什么操作...😓。


***8. Summary***

1.深度卷积可以作为组卷积（groupwise）的一种特例，在常用的轻量级神经网络中（MobileNet系列、shuffleNet、Enet等），Group-wise 和Depth-wise起到了关键性作用。

2.【补充】速度和计算内存代价是衡量轻量型神经网络除了精度之外的重要标准。速度(Speed)是直接指标，但不同设备不好比较， 故以往常用 FLOPs（乘或加的次数）来度量复杂度。但FLOP是一种间接指标，它不能直接作为评判的标准（如Mobilev2和Nasnet相比，他们的Flops相似，但是前者快很多）。例如： 在WRN和Resnet上，WRN的Flops和参数量远大于Resnet情况下，WRN比Resnet快很多。且ResNext比WRN慢很多。[Shufflenetv2](https://arxiv.org/abs/1807.11164)论文中在两种硬件环境中测试四种不同速度和FLOPs的网络结构。观察知道FLOPs不能替代Speed这评判指标。


***Reference***

1.[MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications](https://arxiv.org/abs/1704.04861)

2.[MobileNetV2: Inverted Residuals and Linear Bottlenecks](https://arxiv.org/abs/1801.04381)

3.[Searching for MobileNetV3](https://arxiv.org/abs/1905.02244)

4.[Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385)

5.[Searching for Activation functions](https://arxiv.org/abs/1710.05941)

6.[Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507)

7.[MnasNet: Platform-Aware Neural Architecture Search for Mobile](https://arxiv.org/abs/1807.11626)

8.[MobileNetV3-for-Segmentation](https://github.com/Vipermdl/MobileNetV3-for-Segmentation)

9.[Lightweight-Segmentation](https://github.com/Tramac/Lightweight-Segmentation)

10.[ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design](https://arxiv.org/abs/1807.11164)

11.[ShufflenetV2_高效网络的4条实用准则](https://zhuanlan.zhihu.com/p/42288448)
