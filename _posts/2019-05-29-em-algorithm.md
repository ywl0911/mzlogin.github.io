---
layout: post
title: EM算法
categories: algorithm
description: EM算法
keywords: algorithm
mathjax: true

---

## 1 预备知识
**Jesen不等式**

![Jesen](https://raw.githubusercontent.com/ywl0911/ywl0911.github.io/master/images/posts/database/jensen.gif)

Jesen不等式的内容大概如下：
若$$f(x)$$是区间$$[x_1,x_2]$$上的凸函数，若$$\lambda_1,\lambda_2$$满足$$0\leq\lambda_1,\lambda_2\leq1=\lambda_1+\lambda_2$$，则有不等式:

$$f(\lambda_1x_1+\lambda_2x_2)\leq\lambda_1f(x_1)+\lambda_2f(x_2)$$

当且仅当$$ x_1=x_2$$时取等号，证明可见[这里](https://en.wikipedia.org/wiki/Jensen%27s_inequality)。简单来说就是$$f(E(X)) \leq E(f(X))$$，期望的函数小于等于函数的期望。
Jensen不等式内容虽然不多，但是可以推导出很多其他的不等式。例如不等$$a+b\geq2\sqrt{ab}$$，令$$f(x)=\ln x,\lambda_1=\lambda_2=0.5$$即可推导出来。


**极大似然估计**
极大似然估计是一种确定模型参数值的方法。确定参数值的过程，是找到能极大化模型产生真实观察数据可能性的那一组参数。举个两个例子来讲，就能明白极大似然估计具体是怎么回事。
>例1 抛一枚硬币10次，得到的结果是：正、反、反、正、反、正、正、正、正、正，求抛这枚硬币出现正面的概率。

一般抛硬币只会正面或者反面的概率都是0.5，但是对于这个例子，在10次实验中出现7次正面3次反面，在一定程度上说明了这枚硬币出现正面的概率较大。
假设正面出现的概率为$$p$$，则反面出现概率为$$1-p$$，出现该结果的概率(似然函数)为：

$$L(p)=p^7(1-p)^3$$

求$$L(p)$$的极大值，等价于求$$\ln L(p)$$的极大值，令$$F(p)=\ln L(p)=\ln p^7+\ln(1-p)^3=7\ln p+3\ln(1-p)$$，为求$$F(p)$$的极大值，可对$$F(p)$$求偏导：

$$F^{’}(p)=7\frac{1}{p}-3\frac{1}{1-p}=0 \Rightarrow p=0.7$$

所以$$p=0.7$$时，最有可能出现7正3反的结果。在抛硬币的实验中，硬币出现正面或者反面呈0/1分布，现在生活中更多现象或规律是呈高斯（正态）分布的，见例2。

>例2 假设某个班有$$n$$个人，期末考试成绩分别$$X_1,X_2,...,X_n$$，假设考试成绩$$X$$~$$N(\mu,\sigma^2)$$，求$$\mu，\sigma$$的极大似然估计值。

因为$$X$$~$$N(\mu,\sigma^2)$$，所以$$X$$的概率密度函数为：

$$f(x)=\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$

可以得到似然函数：

$$L(\mu,\sigma^2)=\prod_{i=1}^{n}\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x_i-\mu)^2}{2\sigma^2}}$$

为了方便求解，两边取对数可得到对数似然函数：

$$F(\mu,\sigma^2)=\ln L(\mu,\sigma^2)=\sum_{i=1}^{n}\ln \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x_i-\mu)^2}{2\sigma^2}}
\\
=\sum_{i=1}^{n}\ln \frac{1}{\sqrt{2\pi}\sigma}-\sum_{i=1}^{n}{\frac{(x_i-\mu)^2}{2\sigma^2}}
\\
= -\frac n 2 \ln(2\pi\sigma^2)-\frac 1 {2\sigma^2}\sum_{i=1}^{n}(x_i-\mu)^2$$

对$$F(\mu,\sigma^2)$$求偏导，令$$\frac {\partial F} {\partial \mu} =0$$，$$\frac {\partial F} {\partial \sigma^2} =0$$，得到：

$$\mu=\frac 1 n \sum_{i=1}^{n}x_i= \bar x
\\
\sigma^2=\frac 1 n \sum_{i=1}^{n}(x_i-\bar x)^2$$

至此就根据样本$$X$$求出了$$\mu和σ^2$$，使得$$L(\mu,\sigma^2)$$极大，结果非常直观高斯分布参数的$$\mu和σ^2$$即为随机变量$$X$$的均值和方差。
然后在现实生活中，随机变量可能不只服从一个高斯分布，很有可能是多个高斯分布共同作用的结果，比如一个班同学的身高男生可能服从一个高斯分布$$N(\mu_1,\sigma_1^2)$$，女生服从另一个高斯分布$$N(\mu_2,\sigma_2^2)$$。如何来估计这两个高斯分布的参数呢？

>例3 假设某个班有$$n$$个人，身高分别$$X_1,X_2,...,X_n$$，假设男生和女生身高分别服从$$N(\mu_1,\sigma_1^2)$$和$$N(\mu_2,\sigma_2^2)$$，求$$\mu_1,\mu_2,\sigma_1,\sigma_2$$的极大似然估计值。

对于这个问题我们似乎还是可以沿用之前的极大似然估计的方法，先写出似然函数，然后求偏导令偏导等于0，最后即可求出最优的$$\mu_1,\mu_2,\sigma_1,\sigma_2$$参数值。

已知同学的身高由两个高斯分布产生，那么先假设样本$$x$$由两个高斯分布产生的概率分别为$$\pi_1,\pi_2$$，两个高斯分布的参数先随便给定为$$(\mu_1,\sigma_1),(\mu_2,\sigma_2)$$，那么样本$$x$$产生的先验概率为：

$$p(x)=\sum_{k=1}^2p(k)p(x|k)=\pi_1N(x|\mu_1,\sigma_1)+\pi_2N(x|\mu_2,\sigma_2)$$

其中$$p(k)=\pi_k$$为样本由第$$k$$个高斯分布产生的概率，$$\pi_1+\pi_2=1$$。
可以得到似然函数：

$$L(\mu,\sigma^2)=\prod_{i=1}^{n} p(x_i)=\prod_{i=1}^{n}[\pi_1N(x_i|\mu_1,\sigma_1)+\pi_2N(x_i|\mu_2,\sigma_2)]$$

两边取对数可得到对数似然函数：

$$F(\mu,\sigma^2)=\ln L(\mu,\sigma^2)=\sum_{i=1}^{n}\ln p(x_i)
\\
=\sum_{i=1}^{n}\ln [\pi_1N(x_i|\mu_1,\sigma_1)+\pi_2N(x_i|\mu_2,\sigma_2)]\\
=\sum_{i=1}^{n}\ln
(\pi_1\frac{1}{\sqrt{2\pi}\sigma_1}e^{-\frac{(x_i-\mu_1)^2}{2\sigma^2_1}}+ \pi_2\frac{1}{\sqrt{2\pi}\sigma_2}e^{-\frac{(x_i-\mu_2)^2}{2\sigma^2_2}})
$$

在此我们会发现$$\ln$$里面有+号，无法像单高斯分布那样方便地求出$$\mu_1,\mu_2,\sigma_1,\sigma_2$$的偏导数，所以导致无法求得极大似然估计值。这时候EM算法就能排上用场。

先先验性地假设$$\mu_1=1.75,\sigma_1=10$$，$$\mu_2=1.6,\sigma_2=6$$，即男生的身高服从$$N(1.75,10)$$，女生的身高服从$$N(1.6,5)$$，对应的概率密度函数分别为：

$$f_1(x)=\frac{1}{\sqrt{2\pi}\sigma_1}e^{-\frac{(x-\mu_1)^2}{2\sigma_1^2}} \\
f_2(x)=\frac{1}{\sqrt{2\pi}\sigma_2}e^{-\frac{(x-\mu_2)^2}{2\sigma_2^2}}$$

假设$$\pi_1=\pi_2=0.5$$，即随机变量身高由两个高斯分布产生的概率相同。

抽取一个人$$x_1$$，假设身高为1.9，则可以算出$$x_1$$由两个高斯分布发射出的概率为：

$$f_1(x_1)=f_1(1.9)=0.039\\
f_2(x_1)=f_2(1.9)=0.079$$

归一化后:

$$p_1(x_1)=\frac{f_1(x_1)×\pi_1}{f_1(x_1)×\pi_1+f_2(x_1)×\pi_2}=0.33\\
p_2(x_1)=\frac{f_2(x_1)×\pi_2}{f_1(x_1)×\pi_1+f_2(x_1)×\pi_2}=0.67$$

可以这么认为1.9这个随机变量有0.33的部分由$$N(1.75,10)$$产生，有0.67的部分由$$N(1.6,5)$$产生,可以拆成两个部分：

$$x_1*p_1(x_1)=x_1×0.33=0.627\\
x_1*p_2(x_1)=x_1×0.67=1.273$$

对于$$X$$所有的取值，都可以像$$x_1$$一样算出$$p_1(x_1)，p_2(x_1)$$值，至此就可以进行更新$$\mu_1,\mu_2,\sigma_1,\sigma_2,\pi_1$$,$$\pi_2$$的值了。

$$\mu_1=\frac{1}{\sum_{i=1}^{n}p_1(x_i)}\sum_{i=1}^{n}p_1(x_i)x_i$$

$$\mu_2=\frac{1}{\sum_{i=1}^{n}p_2(x_i)}\sum_{i=1}^{n}p_2(x_i)x_i$$

$$\sigma_1^2=\frac{1}{\sum_{i=1}^{n}p_1(x_i)}\sum_{i=1}^{n}p_1(x_i)(x_i-\mu_1)^2$$

$$\sigma_2^2=\frac{1}{\sum_{i=1}^{n}p_2(x_i)}\sum_{i=1}^{n}p_2(x_i)(x_i-\mu_2)^2$$

$$\pi_1=\frac{\sum_{i=1}^{n}p_1(x_i)}{n}$$

$$\pi_2=\frac{\sum_{i=1}^{n}p_2(x_i)}{n}$$

假设经过上一轮的更新，新的

$$\mu_1=1.72,\mu_2=1.58$$

$$\sigma_1=15,\sigma_2=6$$

$$\pi_1=0.6,\pi_2=0.4$$

即一个人身高服由从$$N(1.72,15)$$，$$N(1.58,6)$$男女两个高斯分布产生出的概率分别为0.6、0.4。
下面重复之前的操作对于$$X$$所有的取值，算出新的$$f_1(x),f_2(x),p_1(x),p_2(x)$$值，更新下一轮$$\mu_1,\mu_2,\sigma_1,\sigma_2,\pi_1,\pi_2$$。至此可以提出该算法的通用表示：

Repeat
{
Given $$\mu_1,\mu_2,\sigma_1,\sigma_2,\pi_1,\pi_2$$

&emsp; $$calculate$$ $$f_1(x_1),f_1(x_2),...,f_1(x_n)$$

&emsp; $$calculate$$ $$f_2(x_1),f_2(x_2),...,f_2(x_n)$$

&emsp; $$calculate$$ $$p_1(x_1),p_1(x_2),...,p_1(x_n)$$

&emsp; $$calculate$$ $$p_2(x_1),p_2(x_2),...,p_2(x_n)$$

Given all $$f_1(·),f_2(·),p_1(·),p_2(·)$$ update $$\mu_1,\mu_2,\sigma_1,\sigma_2,\pi_1,\pi_2$$

&emsp;$$\mu_1=\frac{1}{\sum_{i=1}^{n}p_1(x_i)}\sum_{i=1}^{n}x_ip_1(x_i)$$

&emsp;$$\mu_2=\frac{1}{\sum_{i=1}^{n}p_2(x_i)}\sum_{i=1}^{n}x_ip_2(x_i)$$

&emsp;$$\sigma_1^2=\frac{1}{\sum_{i=1}^{n}p_1(x_i)}\sum_{i=1}^{n}p_1(x_i)(x_i-\mu_1)^2$$

&emsp;$$\sigma_2^2=\frac{1}{\sum_{i=1}^{n}p_2(x_i)}\sum_{i=1}^{n}p_2(x_i)(x_i-\mu_2)^2$$

&emsp;$$\pi_1=\frac{\sum_{i=1}^{n}p_1(x_i)}{n}$$

&emsp;$$\pi_2=\frac{\sum_{i=1}^{n}p_2(x_i)}{n}$$

}until $$\mu_1,\mu_2,\sigma_1,\sigma_2,\pi_1,\pi_2$$ convergence

如果随机变量$$X$$由$$K$$个高斯分布产生，则通用表示为：

Repeat
{

E-Step：通过observed data和现有模型估计参数估计值 missing data；

&emsp;Given $$\mu_k,\sigma_k,\pi_k$$

&emsp;计算：

&emsp;$$f_k(x_i)=N(x_i|\mu_k,\sigma_k)$$

&emsp;$$p_k(x_i)=\frac{\pi_k × f_k(x_i)}{\sum_{j=1}^{K}{\pi_j×f_j(x_i)}}=\frac{\pi_k ×N(x_i|\mu_k,\sigma_k)}{\sum_{j=1}^{K}{\pi_j×N(x_i|\mu_j,\sigma_j)}}$$

M-Step：假设missing data已知的情况下，极大化似然函数。
&emsp;Given all $$f_k(·),p_k(·),g_k(·)$$ update $$\mu_k,\sigma_k,\sigma_k,\pi_k$$

&emsp;$$\mu_k=\frac{1}{\sum_{i=1}^{n}p_k(x_i)}\sum_{i=1}^{n}g_k(x_i)$$

&emsp;$$\sigma_k^2=\frac{1}{\sum_{i=1}^{n}p_k(x_i)}\sum_{i=1}^{n}p_k(x_i)(x_i-\mu_k)(x_i-\mu_k)^T$$

&emsp;$$\pi_k=\frac{\sum_{i=1}^{n}p_k(x_i)}{n}$$
}until $$\mu_k,\sigma_k,\pi_k$$ convergence

以上即为EM算法在高斯混合模型中的应用，直观的变化可以参考如下的变化过程：

![GMM](http://www.csuldw.com/assets/articleImg/2015-12-02-EM_Clustering_of_Old_Faithful_data.gif)

## 2 EM算法导出

观测到$$n$$个相互独立的样本$$\{x(1),…,x(m)\}$$，根据这些数据我们想要拟合模型$$p(x,z)$$，$$z$$为隐变量，如何估计模型的最佳参数呢。
首先写出对数似然函数：

$$L(\theta)=\ln\prod_{i=1}^{n}p(x^{(i)};\theta)
=\sum_{i=1}^{n}\ln p(x^{(i)};\theta)
=\sum_{i=1}^{n}\ln \sum_{z^{(i)}}p(x^{(i)},z^{(i)};\theta)$$

由于$$z$$是一个未知的隐变量，直接求$$\theta$$的偏导数，很难求出最优值。
为此，退而求其次，寻找$$L(\theta)$$的下界，通过极大化下界的方法来求解$$L(\theta)$$的极大值。

$$L(\theta)=\sum_{i=1}^{n}\ln \sum_{z^{(i)}}p(x^{(i)},z^{(i)};\theta)
\\
=\sum_{i=1}^{n}\ln \sum_{z^{(i)}}Q_i(z^{(i)})\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}
\geq
\sum_{i=1}^{n} \sum_{z^{(i)}}Q_i(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}=J(Q,\theta)
$$

因为$$\ln$$为凹函数，根据jesen不等式，有：

$$ \ln E_{z^{(i)}\sim Q_i}\left(\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}\right) \geq E_{z^{(i)} \sim Q_i}\left(\ln \frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}\right)$$

$$J(Q,\theta)$$为$$L(\theta)$$的下界，我们通过调整$$\theta和Q$$来求$$J(Q,\theta)$$极大值，进一步求$$L(\theta)$$的极大值。

在给定的$$\theta^{(i)}$$的情况下，通过调整$$Q(z)$$，使下界$$J(Q,\theta)$$上升与$$L(\theta)$$在$$\theta^{(i)}$$处相切，此时$$J(Q,\theta)$$等于$$L(\theta)$$。因为$$\ln$$为严格的凹函数，根据Jesen不等式的特性，等号成立的条件为当且仅当$$\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}$$为一个常量，即：

$$\frac{p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}=c \Rightarrow$$

$$c=\frac{\sum_{z^{(i)}}{p(x^{(i)},z^{(i)};\theta)}}{\sum_{z^{(i)}}{Q_i(z^{(i)})}}=\frac{p(x^{(i)};\theta)}1={p(x^{(i)};\theta)} \Rightarrow$$

$${Q_i(z^{(i)})}=\frac{p(x^{(i)},z^{(i)};\theta)}{p(x^{(i)};\theta)}=p(z^{(i)}|x^{(i)};\theta)$$

当$${Q_i(z^{(i)})}=p(z^{(i)}|x^{(i)};\theta)$$，$$J(Q,\theta)$$会增加到与$$L(\theta)$$相等。下一步在$${Q_i(z^{(i)})}=p(z^{(i)}|x^{(i)};\theta)$$的情况下，调整参数$$\theta$$来求$$J(Q,\theta)$$的极大值进一步极大化$$L(\theta)$$。

至此，我们推出了在给定参数$$\theta$$后，使下界拉升的$${Q(z)}$$的计算公式，此步就是EM算法的E-step，目的是建立$${L(\theta)}$$的下界的极大值。接下来的M-step，目的是在给定$${Q(z)}$$后，通过调整$$\theta$$，进一步极大化$${L(\theta)}$$的下界。此过程即为EM算法的E-step & M-step，完整的流程如下：

Repeat
{

&emsp;Given $$\theta$$\\

&emsp;&emsp;  $${Q_i(z^{(i)})}:=p(z^{(i)}|x^{(i)};\theta)$$\\

&emsp;Given $${Q_i(z^{(i)})}$$

&emsp;&emsp;  $$\theta:= \arg \max\limits_{\theta} \sum_{i=1}^{n} \sum_{z^{(i)}}Q_i(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta)}{Q_i(z^{(i)})}$$

}until $${L(\theta)}$$ convergence

图示如下：
![GMM](https://images0.cnblogs.com/blog/540980/201307/19012051-44db08d0d7c649a886f6d60acd853fc9.png)

## 3 收敛性证明
假定$$θ^{(t)}$$和$$θ^{(t+1)}$$是EM第$$t$$次和$$t+1$$次迭代后的结果。如果我们证明了$$L(θ^{(t+1)})\geq L(θ^{(t)})$$，也就是说极大似然估计单调增加，那么最终我们就会得到极大似然估计的极大值。
下面来证明，在选定$$θ^{(t)}$$之后，E步的操作为：

$${Q_i^{(t)}(z^{(i)})}:=p(z^{(i)}|x^{(i)};\theta^{(t)})$$

这一步保证了Jesen不等式等号成立：

$$L(\theta^{(t)})=
\sum_{i=1}^{n} \sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta^{(t)})}{Q_i^{(t)}(z^{(i)})}$$

在M步，

$$\theta^{(t+1)}= \arg \max\limits_{\theta} \sum_{i=1}^{n} \sum_{z^{(i)}}Q_i^{(t)}(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta)}{Q_i^{(t)}(z^{(i)})}$$

则有

$$L(\theta^{(t+1)})\geq
\sum_{i=1}^{n} \sum_{z^{(i)}}Q_i(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta^{(t+1)})}{Q_i(z^{(i)})}
 \geq
\sum_{i=1}^{n} \sum_{z^{(i)}}Q_i(z^{(i)})\ln \frac {p(x^{(i)},z^{(i)};\theta^{(t)})}{Q_i(z^{(i)})}
=L(\theta^{(t)})$$

至此就证明了$$L(\theta^{(t+1)})\geq L(\theta^{(t)})$$，说明在每一轮EM之后似然函数$$L(θ)$$单调递增，收敛性得证。