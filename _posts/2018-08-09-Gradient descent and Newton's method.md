---
layout: post
title: "[DL]整理梯度下降法和牛顿法知识点"
excerpt: "详细整理关于梯度下降法和牛顿法具体推导过程，相关知识点。"
date: 2018-08-09 13:34:00
mathjax: true
---

在解优化问题上，常用的方法有梯度下降法和牛顿法，改进的迭代尺度法、拟牛顿法、共轭梯度法等。今天主要整理梯度下降法以及牛顿法两种优化算法。具体内容如下：

**定义**

1.偏导数 <a href="http://www.codecogs.com/eqnedit.php?latex=\tiny&space;\frac{\partial&space;}{\partial&space;x_{i}}f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\tiny&space;\frac{\partial&space;}{\partial&space;x_{i}}f(x)" title="\tiny \frac{\partial }{\partial x_{i}}f(x)" /></a>衡量点 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;x" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;x" title="\small x" /></a> 处只有 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;x_{i}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;x_{i}" title="\small x_{i}" /></a> 增加时 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x)" title="\small f(x)" /></a> 如何变化。

2.梯度是相对一个向量求导的导数： <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a> 的导数是包含所有偏导数的向量，记为 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\bigtriangledown_{x}f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\bigtriangledown_{x}f(x)" title="\small \bigtriangledown_{x}f(x)" /></a>。 梯度的第i个元素是f关于 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;x_{i}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;x_{i}" title="\small x_{i}" /></a> 的偏导数。在多维情况下，临界点是梯度中所有元素都为都为零的点。 

3.方向导数，设函数 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;z=f(x,y)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;z=f(x,y)" title="\small z=f(x,y)" /></a> 在点 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;p(x,y)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;p(x,y)" title="\small p(x,y)" /></a> 的某一邻域内 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;U(P)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;U(P)" title="\small U(P)" /></a> 内有定义，自点p引射线 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;l" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;l" title="\small l" /></a>。设x轴正向到射线 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;l" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;l" title="\small l" /></a> 的转角为 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\varphi" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\varphi" title="\small \varphi" /></a> （逆时针方向<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\varphi" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\varphi" title="\small \varphi" /></a> > 0; 顺时针方向<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\varphi" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\varphi" title="\small \varphi" /></a> < 0)。并设 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;p^{'}(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;p^{'}(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)" title="\small p^{'}(x+\bigtriangleup x,y+\bigtriangleup y)" /></a> 为 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;l" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;l" title="\small l" /></a>上的另一点。且 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;p^{'}\epsilon&space;U(P)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;p^{'}\epsilon&space;U(P)" title="\small p^{'}\epsilon U(P)" /></a>。 我们考虑函数的增量 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)-f(x,y)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)-f(x,y)" title="\small f(x+\bigtriangleup x,y+\bigtriangleup y)-f(x,y)" /></a> 与p <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;p^{'}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;p^{'}" title="\small p^{'}" /></a> 两点间距离 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\rho&space;=\sqrt{\bigtriangleup&space;x^{2}&plus;\bigtriangleup&space;y^{2}}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\rho&space;=\sqrt{\bigtriangleup&space;x^{2}&plus;\bigtriangleup&space;y^{2}}" title="\small \rho =\sqrt{\bigtriangleup x^{2}+\bigtriangleup y^{2}}" /></a> 的比值。当<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;p^{'}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;p^{'}" title="\small p^{'}" /></a> 沿着 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;l" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;l" title="\small l" /></a>趋于p时，如果这个比值的极限存在，则称这极限为函数 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x,y)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x,y)" title="\small f(x,y)" /></a> 在点p沿着方向 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;l" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;l" title="\small l" /></a>的方向导数。记作 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\frac{\partial&space;f}{\partial&space;l}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\frac{\partial&space;f}{\partial&space;l}" title="\small \frac{\partial f}{\partial l}" /></a>， 即：
<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\frac{\partial&space;f}{\partial&space;l}=\lim_{p\rightarrow&space;0}\frac{f(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)-f(x,y))}{\rho&space;}" target="_blank" align="center"><img src="http://latex.codecogs.com/gif.latex?\small&space;\frac{\partial&space;f}{\partial&space;l}=\lim_{p\rightarrow&space;0}\frac{f(x&plus;\bigtriangleup&space;x,y&plus;\bigtriangleup&space;y)-f(x,y))}{\rho&space;}" title="\small \frac{\partial f}{\partial l}=\lim_{p\rightarrow 0}\frac{f(x+\bigtriangleup x,y+\bigtriangleup y)-f(x,y))}{\rho }" /></a>

**梯度下降法**

在 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;u" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;u" title="\small u" /></a> （单位向量）方向的方向导数是函数 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a> 在 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;u" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;u" title="\small u" /></a> 方向的斜率。换句话说，方向是函数 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x&plus;\alpha&space;u)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x&plus;\alpha&space;u)" title="\small f(x+\alpha u)" /></a> 关于 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\alpha" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\alpha" title="\small \alpha" /></a> 的导数（在 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\alpha&space;=0" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\alpha&space;=0" title="\small \alpha =0" /></a> 时取得）。使用链式法则，我们可以看到当 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\alpha&space;=0" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\alpha&space;=0" title="\small \alpha =0" /></a> 时，<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\frac{\partial&space;}{\partial&space;\alpha&space;}f(x&plus;\alpha&space;u)=u^{T}\bigtriangledown_{x}f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\frac{\partial&space;}{\partial&space;\alpha&space;}f(x&plus;\alpha&space;u)=u^{T}\bigtriangledown_{x}f(x)" title="\small \frac{\partial }{\partial \alpha }f(x+\alpha u)=u^{T}\bigtriangledown_{x}f(x)" /></a>。为了最小化 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a> , 我们希望找到使 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a> 下降的最快的方向。计算方向导数：

<div align="center"><a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\underset{u,u^{T}u=1}{min}u^{T}\bigtriangleup&space;_{x}f(x)=\underset{u,u^{T}u=1}{min}\left&space;\|&space;u&space;\right&space;\|_{2}\left&space;\|&space;\bigtriangledown&space;_{x}f(x)&space;\right&space;\|_{2}cos\theta" target="_blank" ><img src="http://latex.codecogs.com/gif.latex?\small&space;\underset{u,u^{T}u=1}{min}u^{T}\bigtriangleup&space;_{x}f(x)=\underset{u,u^{T}u=1}{min}\left&space;\|&space;u&space;\right&space;\|_{2}\left&space;\|&space;\bigtriangledown&space;_{x}f(x)&space;\right&space;\|_{2}cos\theta" title="\small \underset{u,u^{T}u=1}{min}u^{T}\bigtriangleup _{x}f(x)=\underset{u,u^{T}u=1}{min}\left \| u \right \|_{2}\left \| \bigtriangledown _{x}f(x) \right \|_{2}cos\theta" /></a></div>

其中<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\theta" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\theta" title="\small \theta" /></a> 是 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;u" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;u" title="\small u" /></a> 与梯度的夹角。将<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\left&space;\|&space;u&space;\right&space;\|_{2}=1" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\left&space;\|&space;u&space;\right&space;\|_{2}=1" title="\small \left \| u \right \|_{2}=1" /></a>代入，并忽略与 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;u" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;u" title="\small u" /></a> 无关的项，就能简化得到<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\underset{u}{min}cos\theta" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\underset{u}{min}cos\theta" title="\small \underset{u}{min}cos\theta" /></a>。这在 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;u" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;u" title="\small u" /></a> 与梯度方向相反时取得最小。换句话说，梯度向量指向上坡，负梯度向量指向下坡。我们在负梯度方向上移动可以减小<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a>，这被称为梯度下降法。最速下降建议新的点为：
<div align="center"><a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;x^{'}=x-\epsilon&space;\bigtriangledown&space;_{x}f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;x^{'}=x-\epsilon&space;\bigtriangledown&space;_{x}f(x)" title="\small x^{'}=x-\epsilon \bigtriangledown _{x}f(x)" /></a>(<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a>为学习率，是一个确定补偿大小的正标量)</div>

**牛顿法**

我们在使用梯度下降法进行优化时，需要设置 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a> 超参。通常情况下的超参的选择是一个费时费力的工作。如果<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a>设置的过小，整个神经网络的学习过程会变得极其缓慢，过大会导致网络变得震荡，无法取得相对最优的局部极小点。而在某些特定的情况下，我们可以使用二阶导数的性质来判断梯度的变化方向，进而代替梯度下降算法中学习率超参的选择，同时能加快学习过程。具体推到如下：

1.分析：一阶导数将如何随着输入的变化而改变，它表示只基于梯度信息的梯度下降步骤是否会产生如我们预期的那样大的改善，因此它是重要的。我们认为，二阶导数是对曲率的衡量。假设我们的函数 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x)" title="\small f(x)" /></a> （通常情况下，<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x)" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x)" title="\small f(x)" /></a>并不仅仅是二次的，我们可以使用泰勒公式转换成二次函数的近似式子）。如果函数这样的函数具有零二阶导数，那么没有曲率，也就是一条完全平坦的线，仅用梯度就可以预测它的值。我们使用沿负梯度方向大小为 <a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a> 的下降步，当梯度是1时，代价函数将下降<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a>。如果二阶导数是负的，函数曲线向下凹陷，因此代价函数将下降的比<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a> 多。如果二阶导数是正的，函数曲线是向上凹陷，代价函数将下降的比<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;\epsilon" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;\epsilon" title="\small \epsilon" /></a>  少。如下图所示：

<div align="center"><img src="/image/2018-06-26/gradiscent.png" /></div>


泰勒公式展开式如下：
<div align="center"><a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f(x)\approx&space;f(x^{0})&plus;(x-x^{0})^{T}\bigtriangledown&space;_{x}f(x_{0})&plus;\frac{1}{2}(x-x^{0})^{T}H(f)(x^{0})(x-x^{0})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f(x)\approx&space;f(x^{0})&plus;(x-x^{0})^{T}\bigtriangledown&space;_{x}f(x_{0})&plus;\frac{1}{2}(x-x^{0})^{T}H(f)(x^{0})(x-x^{0})" title="\small f(x)\approx f(x^{0})+(x-x^{0})^{T}\bigtriangledown _{x}f(x_{0})+\frac{1}{2}(x-x^{0})^{T}H(f)(x^{0})(x-x^{0})" /></a></div>

因此，我们可以利用泰勒公式，求得该函数的临界点(具体推到[过程](https://blog.csdn.net/linolzhang/article/details/60151623))：
<div align="center"><a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;x^{*}=x^{(0)}-H(f)(x^{(0)})^{-1}\bigtriangledown&space;_{x}f(x^{(0)})" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;x^{*}=x^{(0)}-H(f)(x^{(0)})^{-1}\bigtriangledown&space;_{x}f(x^{(0)})" title="\small x^{*}=x^{(0)}-H(f)(x^{(0)})^{-1}\bigtriangledown _{x}f(x^{(0)})" /></a></div>

如果<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a>是一个正定二次函数，牛顿法只要应用一次上式就能直接跳到函数的最小点。如果<a href="http://www.codecogs.com/eqnedit.php?latex=\small&space;f" target="_blank"><img src="http://latex.codecogs.com/gif.latex?\small&space;f" title="\small f" /></a>不是一个真正二次但能在局部近似为正定二次，牛顿法需要多次迭代应用式子。迭代地更新近似函数和跳到近似函数的最小点可以比梯度下降更快地到达临界点。

**总结**

1.我们在训练神经网络时，寻找的并不是一个全局最优点，而是一个可以接受的局部最优点。

2.在利用梯度下降算法优化时，过大或过小的学习率都不能使得网络得到最优解。

3.在使用Hessian矩阵进行二阶优化时，必须保证Hessian矩阵处处半正定函数，否则梯度的更新方向并不能为最优方向，正因为此，牛顿法并不能在大多数情况下适用。由此提出了很多改进方法，比如拟牛顿法可以看作牛顿法的近似。

**参考文献**

1.花书

2.[牛顿法与Hessian矩阵](https://blog.csdn.net/linolzhang/article/details/60151623)

3.[方向导数与梯度](https://blog.csdn.net/myarrow/article/details/51332421)

4.[梯度下降（Gradient Descent）小结](https://www.cnblogs.com/pinard/p/5970503.html)


















