---
title: "学习率调度器"
date: 2026-09-14
tags: [深度学习, AI, DL]
---

# 学习率调度

在模型训练初期，需要小的学习率warmup，逐步增大学习率；之后需要逐渐衰减学习率

注意：在**每个epoch**训练后，执行`scheduler.step()`（one cycle除外）

## LinearLR

常在Warmup阶段使用，每个epoch逐步提高LR

$$\eta_t = \eta_{max} \frac{t}{T}$$

```py
scheduler = torch.optim.lr_scheduler.LinearLR(
    optimizer, start_factor=1e-3, end_factor=1.0, total_iters=warmup_steps
)
```

## StepLR

用于LR衰减，每过一个间隔将学习率乘以系数

$$\eta_t = \eta_{max} \gamma^{[t/T]}$$

```py
scheduler = torch.optim.lr_scheduler.StepLR(
    optimizer, step_size=3, gamma=0.5
)
```

## 余弦退火 Cosine Annealing

$$\eta_t = \eta_{min} + \frac{1}{2}(\eta_{max}-\eta_{min})(1+\cos(\frac{t\pi}{T}))$$

<figure>
  <img src="/assets/images/deep_learning_basic/CosineAnnealing_pic.webp" alt="CosineAnnealing_pic" />
</figure>

```py
num_epochs = 100
scheduler = CosineAnnealingLR(optimizer, T_max=num_epochs)
for epoch in range(num_epochs):
    train(...)
    validate(...)
    scheduler.step()
```

> 在 *SGDR: Stochastic Gradient Descent with Warm Restarts* 原论文中，每当 $t = T_{max}$ 时，会将 $t$ 设置为0，学习率重新回到最大，模拟重启，用于跳出局部最小。而pytorch实现的这个余弦退火不会重启，学习率的波动是因为余弦函数本身的周期性。如果想要LR不带波动，设置 $T_{max} = \text{num of epochs}$ 即可。

### 带重启 CosineAnnealingWarmRestarts

```py
optimizer = torch.optim.SGD(model.parameters(), lr=0.05)
scheduler = torch.optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer, T_0=20, T_mult=2
)
for epoch in range(100):
    train(...)
    validate(...)
    scheduler.step()
```

第一次开始持续 $T_0$ 然后重启，令 $\eta_t = \eta_{max}$，之后重启周期乘以 $T_{mult}$

<figure>
  <img src="/assets/images/deep_learning_basic/CosineAnnealingWarmRestarts_pic.webp" alt="CosineAnnealingWarmRestarts" />
</figure>

尽管重启的目的是为了跳出局部最小，但是带来了更多代价：需要额外调整参数 $T_0, T_{max}$、在大模型上重启失败的开销。而且数据增强、权重衰减和batch噪声等正则化的方法也可以跳出局部最小

所以现代的大模型依旧使用Warmup+不重启的余弦退火

## ReduceLROnPlateau

之前的学习率调度策略和模型训练状态无关，这个调度器会在平台期降低学习率，模型通常在训练停滞后通过将学习率降低2-10倍来获益，如果在`patience`个epoch内没有看到改进，学习率就会被降低。

```py
optimizer = torch.optim.SGD(model.parameters(), lr=0.1, momentum=0.9)
scheduler = ReduceLROnPlateau(optimizer, "min", factor=0.1, patience=5, min_lr=1e-6)
for epoch in range(10):
    train(...)
    val_loss = validate(...)
# Note that step should be called after validate()
    scheduler.step(val_loss) # 传入val_loss
```

必须在验证集上测试，带来额外开销；LR只会下降

## OneCycleLR

和其他调度器LR主要在衰减不同，OneCycleLR分三个阶段：Warmup让LR先从初始值`max_lr / div_factor`升到最大值`max_lr`；退火Annealing阶段从最大值下降到一个较低的值；最后冷却阶段再急剧下降

在三个阶段中，同时让动量与LR变换相反，实现比标准方法快一个数量级的训练速度

```py
data_loader = torch.utils.data.DataLoader(...)
optimizer = torch.optim.SGD(model.parameters(), lr=1e-4, momentum=0.9)
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=0.01, steps_per_epoch=len(data_loader), epochs=10
)
for epoch in range(10):
    for batch in data_loader:
        train_batch(...)
        optimizer.step()
        scheduler.step() # 在batch中调用
```

**注意在每个batch里调用**

> pytorch默认只实现前两个阶段。
> 
> The default behaviour of this scheduler follows the fastai implementation of 1cycle, which claims that “unpublished work has shown even better results by using only two phases”. To mimic the behaviour of the original paper instead, set `three_phase`=True.

适用于SGD

## 扩展阅读

1. [SGDR: Stochastic Gradient Descent with Warm Restarts](https://arxiv.org/abs/1608.03983)