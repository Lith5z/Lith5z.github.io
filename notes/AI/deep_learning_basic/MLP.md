---
title: "MLP"
date: 2026-09-14
tags: [AI, DL, MLP]
---

# 多层感知器 MLP

训练一个感知器，就是在所有参数构成的空间中训练一个超平面。如果添加多层，依旧是线性变换的组合，结果还是一个线性变换

为了添加非线性，需要在感知器的输出添加一个非线性的[[激活函数]]，最早期的激活函数是阶跃函数：

$$\text{step}(x) = \begin{cases}1, & x \geq 0 \\ 0, & x \lt 0 \end{cases}$$

添加激活函数，在输入层、输出层之外添加隐藏层，这样就成为了多层感知器MLP

根据万能逼近定理，一个含有足够多神经元的单隐藏层MLP，可以以任意精度逼近任意连续函数

## 输出

下面是[[3B1B 多层感知器]]的内容

$$ a^{(1)}_{0} = \sigma(w_{0,0}a^{(0)}_{0}+w_{0,1}a^{(0)}_{1}+\cdots+w_{0,n}a^{(0)}_{n}+b_0) $$

这里 $a^{(1)}_{0}$ 的上标下标代表第1层的第0个神经元。$w_{0,0}$ 的第一个0代表与下层第0个神经元连线，第二个0代表与上一层的第0个神经元连线。

这是对某一个神经元的写法，对于一整层，写成矩阵乘法的形式

$$\begin{pmatrix} w_{0,0} & w_{0,1} & \cdots & w_{0,n} \\ w_{1,0} & w_{1,1} & \cdots & w_{1,n} \\ \vdots & \vdots & \ddots & \vdots \\ w_{k,0} & w_{k,1} & \cdots & w_{k,n} \end{pmatrix} \begin{pmatrix} a^{(0)}_{0} \\ a^{(0)}_{1} \\ \vdots \\ a^{(0)}_{n} \end{pmatrix} + \begin{pmatrix} b_0 \\ b_1 \\ \cdots \\ b_k \end{pmatrix} \xrightarrow{\sigma(x)} \begin{pmatrix} a^{(1)}_{0} \\ a^{(1)}_{1} \\ \vdots \\ a^{(1)}_{k} \end{pmatrix}$$

或

$$\vec{a}^{(1)} = \sigma(\mathbf{W}\vec{a}^{(0)}+\vec{b})$$

左边矩阵的每一行都代表右侧输出神经元的所有权重，整个矩阵代表第0层的权重
第0层有n个神经元，而第1层有k个

<figure>
  <img src="/assets/images/deep_learning/matrixForm_pic.webp" alt="example1" />
</figure>

<figure>
  <img src="/assets/images/deep_learning/total_pic.webp" alt="example2" />
</figure>

## 激活函数

可见[[激活函数]]

## 参数更新

可见[[优化器-梯度下降]]

## 历史

> **1957**：Frank Rosenblatt 发明感知器（Perceptron），能通过简单学习规则调整权重，引发第一次神经网络热潮
> **1969**：Minsky 和 Papert 出版《Perceptrons》，严格证明了单层感知器无法解决 XOR 等线性不可分问题 → 第一次 AI 寒冬
> **1986**：Rumelhart、Hinton 和 Williams 等人发表反向传播算法（Backpropagation），使多层网络的训练成为可能 —— 但受限于当时的算力和数据规模，深层网络难以训练
> **1990s–2000s**：SVM 等浅层方法在性能和理论上均占优，神经网络进入第二次寒冬
> **2006**：Hinton 提出深度信念网络（DBN）和逐层贪心预训练，标志着"深度学习"复兴的开始
> **2012**：AlexNet 在 ImageNet 上大胜传统方法，GPU 并行计算、大规模数据集（ImageNet）和 ReLU/Dropout 等技巧共同开启了现代深度学习时代

## 实例

下面是我写的基于CIFAR10数据集的简单MLP，测试集准确率差不多40%

本来是使用SGD，不添加Dropout和正则化的，但是这样过拟合特别严重，训练集acc到90%多了，测试集还是40%

MLP不适合图像识别任务，就算是对于CIFAR10这样简单的数据集，3\*32\*32的图像意味着输入层需要有3072个神经元，加上隐藏层，参数量会很大，不仅训练慢，在训练数据有限的情况下还会导致过拟合

而且输入图像是把图像拉长为一个一维向量，这样破坏了图像的局部特性

还是得上[[CNN概念]]

```py
"""
一个简单的MLP
"""

import os

# 解决 matplotlib 与 PyTorch 的 OpenMP 运行时冲突
os.environ["KMP_DUPLICATE_LIB_OK"] = "TRUE"

import matplotlib.pyplot as plt
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

# 固定种子、设备
torch.manual_seed(42)
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 超参数
BATCH_SIZE = 128
LEARNING_RATE = 0.001
EPOCHS = 50

# CIFAR-10 数据集的均值和标准差
CIFAR10_MEAN = (0.4914, 0.4822, 0.4465)
CIFAR10_STD = (0.2023, 0.1994, 0.2010)

# 类别名称
CLASSES = (
    "airplane",
    "automobile",
    "bird",
    "cat",
    "deer",
    "dog",
    "frog",
    "horse",
    "ship",
    "truck",
)

# 测试集和训练集的数据增强需要分开
train_transforms = transforms.Compose(
    [
        transforms.RandomCrop(32, padding=4),
        transforms.RandomHorizontalFlip(),
        transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),
        transforms.ToTensor(),  # tensor shape (3,32,32) CHW
        transforms.Normalize(CIFAR10_MEAN, CIFAR10_STD),
        transforms.Lambda(lambda x: x.flatten()),
    ]
)
test_transforms = transforms.Compose(
    [
        transforms.ToTensor(),
        transforms.Normalize(CIFAR10_MEAN, CIFAR10_STD),  # 这个均值和标准差是全局的
        transforms.Lambda(lambda x: x.flatten()),
    ]
)
# 数据集和加载器
trainset = datasets.CIFAR10(
    "D:\\dataset", train=True, transform=train_transforms, download=False
)
testset = datasets.CIFAR10(
    "D:\\dataset", train=False, transform=test_transforms, download=False
)
trainloader = DataLoader(
    trainset, batch_size=BATCH_SIZE, shuffle=True, num_workers=0, pin_memory=True
)
# 测试集没必要shuffle
testloader = DataLoader(
    testset, batch_size=BATCH_SIZE * 2, shuffle=False, num_workers=0, pin_memory=True
)


# 模型
class EasyMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(3072, 512),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(512, 256),
            nn.ReLU(),
            nn.Dropout(0.4),
            nn.Linear(256, 10),
            # 不加softmax，因为交叉熵损失自带
        )

    def forward(self, x):
        x = self.model(x)
        return x


def train_epoch(model, trainloader, criterion, optimizer):
    """训练一个epoch，返回avg loss和acc"""
    current_loss = 0.0
    correct = 0
    total = 0

    model.train()
    for data, label in trainloader:
        data, label = data.to(device), label.to(device)
        optimizer.zero_grad()
        output = model(data)  # output tensor shape (N,10)
        loss = criterion(output, label)
        loss.backward()
        optimizer.step()

        batch_size = label.size(0)
        total += batch_size
        # loss.item() 是 batch 内平均 loss，乘 batch_size 恢复总和
        current_loss += loss.item() * batch_size

        # predicted tensor shape (N)
        # label tensor shape (N)
        _, predicted = output.max(1)  # 按照dim=1取max，输出(value,indices)

        # bool tensor -> tensor shape (1) -> int
        correct += predicted.eq(label).sum().item()

    return current_loss / total, correct / total


def test(model, testloader, criterion):
    """测试整个测试集，返回avg loss和acc"""
    current_loss = 0.0
    correct = 0
    total = 0

    model.eval()
    with torch.no_grad():
        for data, label in testloader:
            data, label = data.to(device), label.to(device)
            batch_size = label.size(0)
            total += batch_size
            output = model(data)
            # loss.item() 是 batch 内平均 loss，乘 batch_size 恢复总和
            current_loss += criterion(output, label).item() * batch_size
            _, predicted = output.max(1)
            correct += predicted.eq(label).sum().item()

    return current_loss / total, correct / total


def main():
    print(f"设备：{device}")

    print(f"训练集数量：{trainset.__len__()}")
    print(f"测试集数量：{testset.__len__()}")

    model = EasyMLP().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.AdamW(model.parameters(), lr=LEARNING_RATE, weight_decay=1e-4)
    scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=EPOCHS)
    print("模型结构：")
    print(model)

    # 训练
    train_loss, train_acc = [], []
    for epoch in range(1, EPOCHS + 1):
        loss, acc = train_epoch(model, trainloader, criterion, optimizer)
        train_loss.append(loss)
        train_acc.append(acc)

        # 每个 epoch 结束后步进 scheduler
        scheduler.step()

        print(
            f"Epoch:{epoch:3d}/{EPOCHS} | Loss:{loss:.4f} | Acc:{acc:.4f} | LR:{scheduler.get_last_lr()[0]:.6f}"
        )

    # 测试
    test_loss, test_acc = test(model, testloader, criterion)
    print(f"\n测试集  Loss:{test_loss:.4f}  Acc:{test_acc:.4f}")

    # 绘图并保存
    fig, (ax_loss, ax_acc) = plt.subplots(1, 2, figsize=(14, 5))
    epochs_range = range(1, EPOCHS + 1)

    # 左图：Loss
    ax_loss.plot(epochs_range, train_loss, color="tab:red")
    ax_loss.set_xlabel("Epoch")
    ax_loss.set_ylabel("Loss")
    ax_loss.set_title("Training Loss")
    ax_loss.grid(True, alpha=0.3)
    # 右图：Accuracy
    ax_acc.plot(epochs_range, train_acc, color="tab:blue")
    ax_acc.set_xlabel("Epoch")
    ax_acc.set_ylabel("Accuracy")
    ax_acc.set_title("Training Accuracy")
    ax_acc.grid(True, alpha=0.3)

    fig.suptitle("CIFAR-10 MLP Training")
    fig.tight_layout()
    plt.savefig("CIFAR10_MLP_training.png", dpi=150)


if __name__ == "__main__":
    main()
```