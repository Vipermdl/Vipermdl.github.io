---
layout: post
title: "[DL]总结深度学习中权重初始化策略"
excerpt: "Deep Learning Best Practices — Weight Initialization。"
date: 2018-11-21 13:34:00
mathjax: true
---

**前言：**

近期在模型调参时发现，一个好的权重初始化方式以及随机种子，对模型的最终训练至关重要。因此，我整理关于DL中随机初始化策略并进行简要总结。

<div style="color:#0000FF" align="center">
<img src="/image/2018-11-21/bp.png" width="860" height="360" /> 
</div>

**1.Initializing all weights to 0**

当我们将所有的权重w都设置为0时，即使在增加非线性损失函数的情况下，在反向传播时，由于损失函数的相同，同一层都会得到相同的梯度值，因此在进行n次迭代时，同一层中的所有的权重仍为一致。这种初始化方式得到结果可能比线性模型结果还要差。注意，我们在此只是强调权重w。

**2.Initializing weights randomly**

通常情况下，我们采用的随机初始化是指满足标准正太分布的随机初始化（np.random.randn(), python),当采用随机初始化时，可能会引起梯度消失和梯度爆炸两个问题。

a)梯度消失

在神经网络中，对于任意的激活函数，当w的梯度值在反向传播时变得越来越小时， 处于前段的网络层便会更新的较慢，进而使得整个网络的收敛变慢。甚至会使得网络停止训练。

注意，使用激活函数中的sigmoid、tanh可能更有可能造成梯度消失，因为当权重较大时，梯度值较小。因此我们更多的采用ReLu激活函数。具体原因略。

梯度消失相比于梯度爆炸来说，更不容易发现，解决起来较为麻烦。

b)梯度爆炸

当你有一个大的权重，随着网络层的逐渐加深，梯度也会变大，这意味着W — ⍺ * dW下降变化剧烈，这可能导致网络在最小值周围震荡甚至一次又一次的超过最佳值，模型将永远不会学习！

梯度爆炸的另外一个常见的影响是梯度的巨大值可能导致数字的越界，从而导致不正确的计算或引入NaN，这可能是产生NaN的一个原因。另外一个产生NaN的原因多半是你的数据有问题，应该着重于检查数据问题。

**3.Best Practices**

a)使用ReLU、leaky ReLU作为激活函数。

b)我们使用启发性初始化方式。

<div style="color:#0000FF" align="center">
<img src="/image/2018-11-21/in.png" width="860"/> 
</div>

c)梯度截断

当梯度超过设定的阈值之后，用设置的阈值或其他方式代替产生的梯度值。For example, normalize the gradients when the L2 norm exceeds a certain threshold – W = W * threshold / l2_norm(W) if l2_norm(W) > threshold.

**note**

本片博客中我们只谈论权重w的影响，bias的初始化影响较小，因为bias的初始化作用于当前层，即使初始化为0，也不回产生梯度爆炸或者消失的影响。


**参考文献**

[1] https://www.coursera.org/learn/deep-neural-network/lecture/RwqYe/weight-initialization-for-deep-networks

[2] Neural networks: training with backpropagation - Jeremy Jordan

[3] A Gentle Introduction to Exploding Gradients in Neural Networks by Jason Brownlee

[4] https://www.quora.com/Why-is-it-a-problem-to-have-exploding-gradients-in-a-neural-net-especially-in-an-RNN

[5] https://github.com/hoya012/deep_learning_object_detection












