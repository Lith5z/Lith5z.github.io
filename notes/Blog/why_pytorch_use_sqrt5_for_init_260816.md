---
title: "为什么PyTorch的默认初始化不是Kaiming/设置参数sqrt5"
date: 2026-08-24
tags: [博客, PyTorch, 参数初始化]
---

# 为什么PyTorch给Linear层的默认初始化不是标准Kaiming/为什么设置参数sqrt5

*最近在学习这方面的内容，发现这个问题网上没有现成的解释，AI给的解释我也不太满意，于是自己去翻了github issue，整理成这份笔记。*

## 理解问题

### 标准的Kaiming初始化

下面是对问题背景的阐述，这两个问题某种意义上是一个问题。如果熟悉背景，可以直接跳到下面看解释部分

神经网络训练中参数的初始化是很重要的一步。如果给所有参数初始化为同一个值，每层的所有神经元都将给出一样的输出，在反向传播时也将得到一样的梯度，这样一整层的神经元相当于退化成一个神经元。

所以需要随机初始化，目前常用的随机初始化方法是**Kaiming初始化**，也就是保证参数的**方差**和标准差是

$$\text{Var}(W) = \frac{2}{n} \implies \text{std}=\sqrt\frac{2}{n}$$

其中 $n$ 通常取 $\text{fan}_\text{in}$（该层输入单元数）

如果从**均匀分布** $U[-a,a]$ 中采样，因为均匀分布的方差是 $\text{Var} = \frac{a^2}{3}$，要满足Kaiming初始化的方差条件，需要让两者方差相等，即

$$\frac{a^2}{3} = \frac{2}{n} \implies a = \sqrt{\frac{6}{n}}$$

也就是说**要满足Kaiming初始化，应当让参数从均匀分布** $U[-\sqrt{\frac{6}{n}},\sqrt{\frac{6}{n}}]$ 中采样。详细推导见结尾参考资料

### PyTorch的实现

但是查阅PyTorch源码发现，对于`nn.Linear`的初始化是这样的：

```Python
def reset_parameters(self) -> None:
    """
    Resets parameters based on their initialization used in ``__init__``.
    """
    # Setting a=sqrt(5) in kaiming_uniform is the same as initializing with
    # uniform(-1/sqrt(in_features), 1/sqrt(in_features)). For details, see
    # https://github.com/pytorch/pytorch/issues/57109
    init.kaiming_uniform_(self.weight, a=math.sqrt(5))
    if self.bias is not None:
        fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
        bound = 1 / math.sqrt(fan_in) if fan_in > 0 else 0
        init.uniform_(self.bias, -bound, bound)
```

这里对权重weight调用了`init.kaiming_uniform_()`，这个函数还有默认参数`nonlinearity='leaky_relu'`未写出，代码里给定的`a=math.sqrt(5)`也就是leaky_relu激活函数的负斜率系数，这里为根号5

这个函数的具体行为是从均匀分布

$$U[-\text{bound},\text{bound}], \quad \text{bound}=\text{gain} \times \sqrt{\frac{3}{\text{fan\_mode}}}$$

里采样，公式中的 $\text{fan\_mode}$ 默认为 $\text{fan\_in}$，也就是我们上文里的 $n$

公式里的 $\text{gain}$ 会调用`torch.nn.init.calculate_gain(nonlinearity, param=None)`来计算。对于上面的代码，默认给定的`nonlinearity='leaky_relu'`，给定负斜率系数为根号5，所以增益 $\text{gain}$ 为

$$\text{gain} = \sqrt{\frac{2}{1+\text{negative\_slope}^2}} = \sqrt\frac{2}{1+5} = \sqrt{\frac{1}{3}}$$

再代入

$$\text{bound}=\text{gain} \times \sqrt{\frac{3}{\text{fan\_mode}}} = \sqrt{\frac{1}{3}} \times \sqrt{\frac{3}{n}} = \sqrt{\frac{1}{n}}$$

综上，**PyTorch对于权重调用的**`init.kaiming_uniform_(self.weight, a=math.sqrt(5))`，**等价于从均匀分布** $U[-\sqrt{\frac{1}{n}},\sqrt{\frac{1}{n}}]$ 中采样

这里忽略了偏置bias，实际上偏置虽然没有使用`kaiming_uniform_`，但它和权重实际采样的分布相同，这点可以从代码`init.uniform_(self.bias, -bound, bound)`里看出

### 问题所在

我们刚刚简单推导过，标准的Kaiming初始化，应当从 $U[-\sqrt{\frac{6}{n}},\sqrt{\frac{6}{n}}]$ 中采样；但是PyTorch的实现却是从 $U[-\sqrt{\frac{1}{n}},\sqrt{\frac{1}{n}}]$ 中采样的

这就是我们开篇的问题

- 为什么PyTorch给Linear层的默认初始化不是标准Kaiming初始化？
- 或者说，设置一个参数sqrt5的意义是什么？
- 其实我还看到过第三种问法：为什么PyTorch的标准差只有标准Kaiming初始化的40%左右？

可以计算PyTorch实现的初始化的标准差。对于 $U[-\sqrt{\frac{1}{n}},\sqrt{\frac{1}{n}}]$，代入均匀分布的方差公式，得到

$$\text{Var} = \frac{a^2}{3} = \frac{1}{3n} \implies \text{std} = \sqrt{\frac{1}{3n}}$$

与Kaiming初始化的标准差 $\text{std}=\sqrt\frac{2}{n}$ 作比例

$$\frac{\sqrt{\frac{1}{3n}}}{\sqrt\frac{2}{n}} = \frac{1}{\sqrt{6}} \approx 40.82 \%$$

## 一种解释

我看到最多的解释是**向后兼容**，也就是在PyTorch的前身，Lush/Senna/Torch5/Torch7都在使用 $U[-\sqrt{\frac{1}{n}},\sqrt{\frac{1}{n}}]$ 的初始化方法，所以PyTorch最初使用这个初始化而非Kaiming初始化

最初PyTorch的代码是这样的

```Python
def reset_parameters(self):
    stdv = 1. / math.sqrt(self.weight.size(1))
    self.weight.data.uniform_(-stdv, stdv)
    if self.bias is not None:
        self.bias.data.uniform_(-stdv, stdv)
```

后来PyTorch的默认初始化改为调用`kaiming_uniform_()`初始化，具体计算可以看之前解释PyTorch实现的部分

所以这个根号5的参数是为了让修改前后的效果一致

## 一种更深入的解释

但是上一种解释里，没有说明为什么不用标准的Kaiming初始化，这个sqrt5也是为了兼容硬凑出来的，我觉得应该有更深层的原因

查阅GitHub仓库，我找到了两个和本问题强相关的issue，而且都来自同一个人soumtih的回复（加粗是我自己加的标注，原本没有）

[GitHub issue#15314](https://github.com/pytorch/pytorch/issues/15314)

> the code refactor from jramseyer changes the default pytorch initialization from manually initializing the weights **by calling random number generator function uniform to using torch.nn.init**.kaiming -- but it **wanted to have the same end-result in weights**, because we wanted to preserve backward-compatibility. So the sqrt(5) is nothing more than giving the code the same end-result as before.
>
> **The initialization itself comes from torch7 and torch5 and is a modified version of initialization fro Lecun'98 Efficient Backprop**. This post gives more context: https://plus.google.com/106447253626219410322/posts/RZfdrRQWL6u

这条回复的前半部分和第一种解释一样，是为了达成同样的效果才设置了这个参数；后半部分和另一条issue里的回复

[GitHub issue#57109](https://github.com/pytorch/pytorch/issues/57109)

> one thing worth noticing here is that by default, **we don't multiply the stdv with sqrt(3)**.
> The reason we don't do that because **Collobert at al. in some historical past have figured out that this not-multiplying and having a slight gain** in the uniform distributions heuristically works better. This is not recorded in literature anywhere but has been recorded in code since Lush, Senna, Torch5, Torch7 and now PyTorch which is somewhat unfortunate. A detailed discussion of this was recorded on **Google Plus**, which is now defunct and that discussion has been erased from the internet. However, I've revived a copy of that discussion here: https://soumith.ch/files/20141213_gplus_nninit_discussion.htm

都指向一篇2014年在Google+ Deep Learning Community上的讨论，大概是说这其实是一个bug，但是结果得到了一个轻微的提升，于是就保留在了代码中，一直延续到现在的PyTorch里

在这篇讨论里，Soumith Chintala提到

> A few months ago we had a small post here discussing different **weight initializations**, and I remember +Sander Dieleman and a few others had a good discussion. It is fairly important to do good weight initializations, as the rewards are non-trivial.
> 
> For example, AlexNet, which is fairly popular, from Alex's One Weird Trick paper, converges in 90 epochs (using alex's 0.01 stdv initialization).
> 
> I retrained it from scratch using the weight initialization from Yann's 98 paper, and **it converges to the same error within just 50 epochs**, so technically +Alex Krizhevsky could've rewritten the paper with even more stellar results (training Alexnet in 8 hours with 8 GPUs).
> 
> In fact, more interestingly, just by doing good weight initialization, I **even removed the Local Response Normalization layers in AlexNet with no drop in error**.

他的初始化参考了LeCun的论文*Efficient BackProp*的初始化，需要先满足两个条件

- 训练集经过标准化
- 使用 $f(x) = 1.7159 \tanh(\frac{2}{3}x)$ 作为激活函数

然后从满足均值为0同时 $\text{std}=\sqrt{\frac{1}{n}}, \quad n = \text{fan}_\text{in}$ 的均匀分布中采样，要满足这个条件，也就是从

$$U[-\sqrt{3} \times \sqrt{\frac{1}{n}},\sqrt{3} \times \sqrt{\frac{1}{n}}] = U[-\sqrt{3} \times \text{std}, \sqrt{3} \times \text{std}]$$

里采样（计算同理），边界是标准差的根号3倍

但是他的代码写错了（由Sander Dieleman指出），写成了 $U[-\text{std}, \text{std}]$，少乘了一个根号3

（FYI，少乘一个根号3之后，就正好是现在PyTorch的标准差是 $\sqrt{\frac{1}{3n}}$。而Kaiming初始化的标准差是 $\sqrt\frac{2}{n}$）

Sander Dieleman回复的原话是

> Actually, after reading the code that +Soumith Chintala linked more carefully, the distribution that the initial values are drawn from doesn't have the specified stddev at all, so **it's not actually implementing the initialization from LeCun '98**.
> 
> **In the code, the distribution is Uniform[-stddev, stddev], but to get the desired standard deviation, this should actually be Uniform[-sqrt(3)\*stddev, sqrt(3)\*stddev]**.
> 
> With that factor of sqrt(3) missing there isn't really a theoretical justification for this initialization. **But clearly it seems to work well**, from Soumith's experience, so what's going on there?

尽管代码写错了，但是最后的效果又很好，Soumith Chintala回复

> Ah yes **its a good bug**. Apparently it was a bug to not multiply by sqrt(3) but it **helped with some gradient scaling issues across layers**, so **it was left in there** as it works slightly better in practice

因为没乘根号3反而对梯度在层间的缩放问题有帮助，这个bug被保留了

这篇讨论想按着Efficient BackProp论文的初始化操作，但是少乘了一个根号3，反而带来了更好的效果，所以这个初始化一直保留到现在的PyTorch（应该还是为了向后兼容）；而在当时，Kaiming初始化可能还没提出或者没被广泛运用，这大概就是问题的答案

### 那么，为什么这样效果更好？

*下面的内容仅为我的个人猜测，我还在学习这方面的内容，水平有限，欢迎批评指正。*

既然PyTorch是为了兼容没有改掉这个良性bug，那么为什么在当时，少乘的根号3反而有更好的效果呢？

理论上说，$\sqrt{\frac{1}{3n}}$ 的标准差确实偏小了些。对于当时用的tanh和sigmoid（也许是），应该使用Xavier初始化要求的 $\sqrt{\frac{1}{n}}$，以保证激活值和梯度在层间传播保持稳定，既不饱和也不爆炸

我猜当时的网络层数还不深，更小的方差意味着预激活值的方差也更小，预激活值就会更多的落在0附近，这可能减弱了预激活值过大/过小（饱和）带来的梯度消失现象，也就是所谓的"helped with some gradient scaling issues across layers"

而对于后来更深层的、使用ReLU激活函数的网络，PyTorch的这个默认初始化就会带来问题了，这时候需要自己手动调用以实现标准的Kaiming Uniform

这还不是一个完全满意的解释，也暂时没找到当时Soumith Chintala的源码，之后有机会可能会继续补充qwq

## 参考资料

1. 关于Kaiming初始化的数学推导：[[论文阅读] Delving Deep into Rectifiers:Surpassing Human-Level Performance on ImageNet Classification](https://zhuanlan.zhihu.com/p/1999932426844125029)
2. Kaiming初始化论文原文：[Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification](https://arxiv.org/abs/1502.01852)
3. PyTorch关于Linear层默认初始化的源码：[GitHub linear.py 行117](https://github.com/pytorch/pytorch/blob/8f988c9c6b3586efbc00a981d9d8cac11f26bcdb/torch/nn/modules/linear.py#L117)
4. PyTorch关于nn.init的官方文档：[torch.nn.init](https://docs.pytorch.org/docs/2.13/nn.init.html)
5. 一个类似的知乎问题里：[作者：西伯利亚大恶龙 Pytorch中Linear层默认的初始化方式中a=sqrt(5)的原理是什么？](https://www.zhihu.com/question/405929936/answer/1786694422)
6. 一条关于这个问题的issue下的回复里给出了真正的历史原因：[GitHub issue#57109](https://github.com/pytorch/pytorch/issues/57109)
7. 另一条issue下也指向同一个网站：[GitHub issue#15314](https://github.com/pytorch/pytorch/issues/15314)
8. 两条issue指向的网站备份：[Google Plus论坛备份](https://soumith.ch/files/20141213_gplus_nninit_discussion.htm)
9. 实际上应该是这个打不开的网站：[原网站](https://plus.google.com/106447253626219410322/posts/RZfdrRQWL6u)
10. [Efficient BackProp](http://yann.lecun.com/exdb/publis/pdf/lecun-98b.pdf)