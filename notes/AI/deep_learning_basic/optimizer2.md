---
title: "优化器 自适应"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 自适应学习率

## AdaGrad

即Adaptive Gradient Algorithm

对于每一个参数，都额外维护梯度的平方累积 $G_t = G_{t+1} + \nabla L(\theta_t)^2$，得到调整后的学习率

$$\Delta \theta = - \frac{\eta}{\sqrt{G_t+\epsilon}} \cdot \nabla L(\theta_t)$$

更新频繁的参数积累的梯度大，学习率降低，防止更新过度；更新稀少的参数反之

> 这里出现在分母的 $\epsilon$ 是为了防止除以0，一般取1e-8。
>
> 至于为什么 $\epsilon$ 在根号内，是因为在训练初期 $v_t$ 较小的时候，对 $\epsilon$ 开根号可以避免学习率被放大太多。

- 对全局学习率的敏感度低
- 适合稀疏数据（如NLP）
- $G_t$ 一直积累，学习率越来越小，可能在模型还没有充分收敛的时候，学习率已经趋于0
- 理论上两倍显存开销：需要为每个参数储存 $G_t$

## 均方根传播 RMSProp

把平方梯度的累积改成滑动平均，类似动量的累积

$$v_t = \beta v_{t-1} + (1-\beta) \nabla L(\theta_t)^2$$

由于历史的平方梯度 $v_{t-1}$ 乘以了衰减因子 $\beta$，学习率不会单调衰减到0，然后更新参数

$$\Delta \theta = - \frac{\eta}{\sqrt{v_t+\epsilon}} \cdot \nabla L(\theta_t)$$

类似[[优化器-梯度下降和动量]]展开 $v_t$ 来看：

$$v_t = (1 - \beta) \sum_{i=1}^{t} \beta^{t-i} \cdot \nabla L(\theta_t)^2$$

这意味着：

- 当前梯度 $\nabla L(\theta_t)$ 的权重是 $(1 - \beta) \approx 0.1$
- 上一个梯度 $\nabla L(\theta_{t-1})$ 的权重是 $(1-\beta)\beta \approx 0.09$
- 过去第 10 步的梯度权重是 $(1-\beta)\beta^{10} \approx 0.035$

通常 $\beta=0.9$ 时，相当于只关注**最近 10 ~ 20 个 batch 的梯度大小**

- 更适应非凸损失函数（如深度学习）的优化
- 理论上两倍显存开销：需要为每个参数储存 $v_t$

## 自适应矩估计 Adam/AdamW

即Adaptive Moment Estimation

在RMSProp的基础之上引入动量

$$m_t = \beta_1 m_{t-1} + (1-\beta_1) \nabla L(\theta_t)$$

$$v_t = \beta_2 v_{t-1} + (1-\beta_2) \nabla L(\theta_t)^2$$

其中动量 $m_t$ 和 二阶动量 $v_t$ 也就是一阶矩估计和二阶矩估计，$m_t$ 用于动量法，$v_t$ 用于缩放学习率；超参数一般取 $\beta_1 =0.9, \beta_2 = 0.999$

然后再进行无偏修正：

$$\hat{m}_t = \frac{m_t}{1 - \beta_1^t}$$

$$\hat{v}_t = \frac{v_t}{1 - \beta_2^t}$$

由于 $m_0$ 和 $v_0$ 初始化为 0，在训练初期，$m_t$ 和 $v_t$ 会严重偏向 0。比如 $t=1$ 时，$m_1 = 0.1 \nabla L(\theta_t)$，只有真实梯度的十分之一，而 $\hat{m}_1 = m_1 / 0.1 = \nabla L(\theta_t)$，修正后等于真实梯度

随着 $t$ 增大，分母趋近于 1，修正项逐渐失效，退化为普通移动平均

参数更新公式是：

$$\Delta \theta = - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \cdot \hat{m}_t$$

> 注意 Adam 的公式里，$\epsilon$ 在根号外面，和RMSProp不同。
> 
> 因为 Adam 做了**偏差修正**，$\hat{v}_t$ 在训练初期不会像原始 RMSProp 的 $v_t$ 那样出现极端接近于 0 的情况，所以把 $\epsilon$ 加在外面也不会有问题。

如果把自适应学习率 $\frac{\eta}{\sqrt{\hat{v}_t} + \epsilon}$ 看成一个常量，更新项和动量法 $- \eta \cdot v_t$ 是一致的

如果让一阶矩 $\hat{m}_t$ 不用滑动平均而是直接等于当前梯度 $\nabla L(\theta_t)$，更新项和RMSProp的 $\frac{\eta}{\sqrt{v_t+\epsilon}} \cdot \nabla L(\theta_t)$ 是一致的

### AdamW

在SGD里

- L2正则化：在损失函数后加一个 $\frac{\lambda}{2}||\theta||^2$ 的惩罚项，变为 $L' = L + \frac{\lambda}{2}||\theta||^2$，从而梯度的计算公式变为 $\nabla L' = \nabla L + \lambda ||\theta||$，影响后续参数更新
- 权重衰减：先将参数乘以一个小于1的系数，再进行参数更新，即 $\theta_{t+1} = (1 - \eta \lambda)\theta_t - \eta \nabla L$

两种实现方式在数学上是等价的，可见[[正则化]]

在pytorch的SGD里，是通过L2正则化来实现权重衰减的，具体来说，是在反向传播计算完梯度后，直接执行`grad = grad + weight_decay * param`，然后参数更新正常进行，不单独乘系数

Adam的参数更新公式和SGD不同，所以两者实现不等价。但是在旧版Adam里，依旧沿用了SGD直接在梯度上添加的做法，导致 $v_t, m_t$ 都受到影响

- 对于**历史梯度很大**的参数（如高频词），$\hat{v}_t$ 很大，有效学习率 $\eta / \sqrt{\hat{v}}$ 很小。此时，梯度里加的惩罚项 $\lambda \theta$ 被这个**很小的有效学习率**缩放了，导致正则化效果极弱。
- 对于**历史梯度很小**的参数（如低频词），$\hat{v}_t$ 很小，有效学习率很大。此时，惩罚项被放大，导致正则化效果极强。

对学习率的缩放影响了权重衰减的效果，破坏了正则化的作用

AdamW相对于Adam做的修正有两点：

1. 计算梯度不添加`weight_decay * param`这一项，让梯度只来自于损失函数
2. 参数更新时，执行 $\theta_t \leftarrow \theta_{t-1} - \eta (\hat{m}_t / (\sqrt{\hat{v}_t} + \epsilon) + \lambda \theta_{t-1})$，也就是改为使用标准的权重衰减写法，不走L2正则化路径

> 关于到底是先对旧参数乘衰减系数，还是先根据梯度更新旧参数：
> 
> 在AdamW的论文原文里，是 $\theta_t \leftarrow \theta_{t-1} - \eta (\hat{m}_t / (\sqrt{\hat{v}_t} + \epsilon) + \lambda \theta_{t-1})$，展开重组之后也就是先执行权重衰减，即乘以 $(1-\eta \lambda)$，再更新参数（原文的公式还多一个学习率调度参数，这里略去）。
> 在PyTorch的源码里，AdamW实现方式也是先乘以衰减系数，再更新参数。AdamW继承Adam类，在`adam.py`里可以看到（##是我添加的注释）：
> ```py
> if weight_decay != 0:
>     if decoupled_weight_decay: ## adamW走这个分支
>         # Perform step weight decay
>         param.mul_(1 - lr * weight_decay)
>     else: ## 原始adam走这里，直接修改了梯度grad
>         # Nested if is necessary to bypass jitscript rules
>         if differentiable and isinstance(weight_decay, Tensor):
>             if weight_decay.requires_grad:
>                 grad = grad.addcmul_(param.clone(), weight_decay)
>             else:
>                 grad = grad.add(param, alpha=weight_decay)
>         else:
>             grad = grad.add(param, alpha=weight_decay)
> ## 后面是参数更新的代码
> ```
> SGD的实现和原始Adam的类似，这里不展示了。
> 
> 不过，交换顺序的话，差别也只是一个小量 $\eta^2 \lambda \text{Update}$，可以忽略。

- 结合了动量法和RMSProp的优点
- 在Adam中，最优的L2正则化系数 $\lambda$ 高度依赖于学习率 $\alpha$ 的设置。AdamW将两者解耦，使超参数调节更容易
- AdamW通常能带来更好的泛化性能，成为现代大模型的优化器
- 理论上3倍显存占用，因为额外储存了 $v_t,m_t$。如果算上临时梯度，理论上是四倍

## 扩展阅读

1. [关于优化器的PyTorch文档](https://docs.pytorch.org/docs/2.13/optim.html)
2. [PyTorch的adam.py源码line416](https://github.com/pytorch/pytorch/blob/v2.13.0/torch/optim/adam.py#L416)
3. [Hugging Face关于显存占用的问题的一篇博客](https://huggingface.co/blog/train_memory)