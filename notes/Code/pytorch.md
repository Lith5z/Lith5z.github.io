---
title: "语法 PyTorch"
date: 2026-09-14
tags: [编程, 深度学习, AI]
---

# PyTorch

import torch

可以参阅[torch doc](https://docs.pytorch.org/docs/2.13/nn.html)

## tensor张量

`torch.tensor`

tensor类包装了梯度等训练需要的信息，和np数组的使用基本一致

创建时的参数`.requires_grad`
bool，决定pytorch是否要对这个张量构建运算图，从而自动求导

`.shape` `.dtype` `.device`

`.item()` 转为标量

### 创建

```py
tensor1 = torch.tensor([[1, 2, 3],
                        [4, 5, 6]])
```

`torch.zeros()` `torch.ones()` `torch.randn` 传入shape创建

### 变形

```py
# (B, 3, 32, 32) -> (B, 3072)，下面的三个写法均可
x.view(x.size(0), -1) # 要求内存连续，更快些
x.reshape(x.size(0), -1)
x.flatten(1)  # 从第1维开始展平
```

### 读取图片

1. PIL对象：用`Image.open("path")`
2. np数组：可以把PIL对象用`np.array()`转换；但是opencv的方法更快`img = cv2.imread("path")`
3. tensor：需要transforms实例化一个ToTensor类，再调用

注意，PIL和np的shape都是HWC，数据范围是[0,255]；而tensor是CHW，范围是[0,1]；三者都可以通过[][][]来访问某个像素的值

## Dataset

`from torch.utils.data import Dataset`

创建自己的数据集，需要重载下面的函数：

- `__getitem__(self, idx)` 按照索引读取一个样本和标签，实际使用可以用[]直接读取
- `__len__(self)` 返回数据集的大小

下载的数据集按照文件夹的形式保存在本地，但是getitem需要文件的idx索引，这需要把文件对应到索引上

下面的操作根据不同数据集的形式而不同，以蚂蚁蜜蜂数据集为例：
```
hymenoptera_data:
├─train
│ ├─ants
│ │ ├─0013035.jpg
│ │ └─...
│ └─bees
└─val
  ├─ants
  └─bees
```

label是图片所在文件夹名，各个图片样本的名字是乱码

```py
from torch.utils.data import Dataset
import os
from PIL import Image

class MyData(Dataset):
    def __init__(self, root_dir, label_dir):
        super().__init__()
        self.root_dir = root_dir
        self.label_dir = label_dir
        # 按照系统路径的格式连接
        self.path = os.path.join(self.root_dir, self.label_dir)
        # 把路径下的各个文件映射为index-file的列表
        self.img_path = os.listdir(self.path)

    def __getitem__(self, index): # 就是重载[]
        img_name = self.img_path[index]
        img_item_path = os.path.join(self.root_dir, self.label_dir, img_name) # 拼接为完整的图片路径
        img  = Image.open(img_item_path)
        label = self.label_dir # 标签正好是文件夹名
        return img, label
    def __len__(self):
        return len(self.img_path) # 直接返回列表的长度
    
ant_dataset = MyData("D:\\dataset\\hymenoptera_data\\train", "ants")
# 注意windows用\\作为分隔
img0,_ = ant_dataset[0] # getitem重载[]
img0.show()
```

如果有两个MyData实例，可以直接相加，顺序按照相加顺序进行排列

torchvision中也提供了很多数据集，有现成的dataset类
```py
from torchvision import transforms,datasets
test_dataset = datasets.CIFAR10("D:\\dataset", # 数据集的文件夹所在的目录
                                train=True,
                                transform=transforms.ToTensor(),
                                download=False)
```

## transforms

`from torchvision import transforms`

对图片进行一些变换

```py
img = Image.open("./test.jpg")
tensor_trans = transforms.ToTensor() # 实例化
tensor1 = tensor_trans(img) # 调用
```

常用的类：

- `ToTensor` 将PIL对象或者np数组转化为tensor
- `Normalize` 按照给定均值和标准差，对各个通道进行归一化
- `Resize` 输入PIL输出PIL
- `RandomCrop` 在图片上随机裁剪下来给定的大小
- `RandomErasing` 随机擦除
- `Compose` 把多个变换打包，便于直接调用。注意各步的**输出输入类型匹配**

一般来说，在ToTensor之前进行数据增强操作，之后进行Normalize等操作

```py
trans = transforms.Compose([
    transforms.Resize((32,32)), # 输入PIL输出PIL
    transforms.ToTensor(),
    transforms.Normalize([0,0,0], [1,1,1]) # 均值和标准差，每个都是三个通道
])

img = Image.open("./test.jpg")
tensor1 = trans(img)
```

在Dataset中往往需要指定transform参数

顺序约定：所有 PIL/几何变换 -> ToTensor -> Normalize -> Erasing

## DataLoader

`from torch.utils.data import DataLoader`

用于从数据集中加载每一个batch所需要的数据，常用参数：

- `dataset` 传入Dataset类的实例，需要是tensor
- `batch_size`
- `shuffle` bool，每个epoch是否要打乱batch的选定
- `num_workers` 所用进程数量，默认为0
- `pin_memory` 如果开启可以加速cpu到gpu的数据传输
- `drop_last` bool，数据集不能被batch_size除尽的时候，最后一个剩余的batch要不要直接舍去

```py
test_dataset = datasets.CIFAR10("D:\\dataset",
                                train=True,
                                transform=transforms.ToTensor(),
                                download=False)
loader = DataLoader(test_dataset,batch_size=4,shuffle=True)

for data in loader:
    imgs,labels = data
    # batch_size是4，一次加载4个样本
    print(imgs.shape) # torch.Size([4, 3, 32, 32]) 4个图片，CHW
    print(labels) # tensor([5, 3, 2, 2]) 四个图片的label
    exit()
```

`num_workers`参数在windows下有点问题，如果要设置非0值，dataset/dataloader的创建必须在main里，不能在代码的顶层，比如：

```py
def main():
    print(f"设备：{device}")

    # Windows 的多进程采用 spawn 方式启动，子进程会重新 import 本模块，
    # 若 DataLoader 在模块级创建，会在每个 worker 中重复执行（浪费内存甚至递归报错），
    # 因此必须放在 if __name__ == "__main__" 保护链路内创建
    trainset = datasets.CIFAR10(
        DATA_PATH, train=True, transform=train_transforms, download=False
    )
    testset = datasets.CIFAR10(
        DATA_PATH, train=False, transform=test_transforms, download=False
    )
    trainloader = DataLoader(
        trainset,
        batch_size=BATCH_SIZE,
        shuffle=True,
        num_workers=NUM_WORKERS,
        pin_memory=True,
        persistent_workers=NUM_WORKERS > 0,  # 避免每个 epoch 重新 spawn worker
    )
    # 测试集没必要shuffle
    testloader = DataLoader(
        testset,
        batch_size=BATCH_SIZE * 2,
        shuffle=False,
        num_workers=NUM_WORKERS,
        pin_memory=True,
        persistent_workers=NUM_WORKERS > 0,
    )
    # 剩下的是实例化模型等代码...

if __name__ == "__main__":
    main()
```

## torch.nn

`import torch.nn as nn`
`import torch.nn.functional as F` 封装前的原始API

### nn.Module

所有的神经网络都要继承nn.Module作为框架，并重写方法：

- `__init__` 在这里写好每一层是什么变换，之后在forward里调用
- `forward(self,x)` 如何对一个batch的数据进行前向传播

forward方法是对`()`的重载，之后可以直接写`output = model(input)`

实例化的模型可以直接print，查看模型的结构

### 连接层

- `Linear` 等线性层
    - `in_features` `out_features`
- `Conv2d` 二维卷积层，同理还有1d 3d，具体参数见CNN笔记
- `MaxPool2d` `AdaptiveAvgPool2d` 二维池化层
- `BatchNorm2d` 二维正则化层，对过程中每层的结果进行标准化，加速收敛
    - `num_features` 输入通道数
- `Dropout`
- `ReLU`等各类激活函数
    - `inplace`参数决定是要写成`x = relu(x)`还是直接`relu(x)`，也就是是否原地操作

线性层（得到预激活值） - 归一化层 - 激活函数层 - Dropout

### Sequential

简化前向传播的代码

```py
class Test(nn.Module):
    def __init__(self):
        super(Test, self).__init__()
        self.model = nn.Sequential(
            nn.Conv2d(1, 20, 5),
            nn.ReLU(),
            nn.Conv2d(20, 64, 5),
            nn.ReLU()
        )
    def forward(self, x):
        x = self.model(x)
        return x
```

但是，如果网络有跳跃的连接（比如ResNet的残差连接），就不能这样写了

### 如何验证shape

```py
X = torch.randn(1, 3, 32, 32)
model = MyCNN()

for layer in model.model:
    X = layer(X)
    print(layer.__class__.__name__, "output shape:\t", X.shape)
```

### nn.init

构建好的模型，对于Linear层、Conv2d层，会自动使用Kaiming初始化权重

权重使用`init.kaiming_uniform_(self.weight, a=math.sqrt(5))`，计算之后实际上就是在 Uniform(-1/√fan_in, 1/√fan_in) 分布中采样

具体代码见[pytorch源码linear.py](https://github.com/pytorch/pytorch/blob/main/torch/nn/modules/linear.py)

```py
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

> 这可能不是标准的Kaiming初始化。如果是ReLU激活函数，这里的分布应该再乘增益 根号2，而pytorch默认没有，导致`nn.Linear`默认初始化的权重标准差只有针对ReLU的标准Kaiming初始化的40%左右。浅层网络的影响不大，对于**深层网络，需要自行设置正确的gain数值**。

#### 手动初始化

不同的激活函数需要不同的初始化公式，pytorch的做法是提供一个增益因子gain

```py
gain = nn.init.calculate_gain(
    "leaky_relu", 0.2
)  # leaky_relu with negative_slope=0.2
```

会返回不同激活函数的推荐数值，比如Linear线性为1，sigmoid为1，ReLU为根号2等

然后可以调用以下的初始化：

- `.zeros_()` `.ones_()`可以手动初始化为全0、全1
- 对于sigmoid和tanh使用的Xavier，要给定参数gain的数值，默认为1
- 对于ReLU族使用的Kaiming，需要指定`nonlinearity`也就是所用的激活函数（默认leaky_relu，这时需要设置斜率`a`），pytorch会调用`calculate_gain`计算增益

具体例子是

```py
import torch.nn as nn
import torch.nn.init as init

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

        # 在这里调用初始化
        self._initialize_weights()

    def _initialize_weights(self):
        # 定义初始化规则
        def init_fn(m):
            if isinstance(m, nn.Linear):
                # 标准的 Kaiming 均匀初始化（针对 ReLU）
                # a=0, mode='fan_in', nonlinearity='relu' 对应的 gain 自动取 sqrt(2)
                init.kaiming_uniform_(m.weight, a=0, mode='fan_in', nonlinearity='relu')
                # 偏置初始化为 0
                if m.bias is not None:
                    init.constant_(m.bias, 0)
        
        # apply 会递归地遍历 self.model 的所有子模块，对每个子模块调用 init_fn
        self.model.apply(init_fn)

    def forward(self, x):
        x = self.model(x)
        return x
```

#### 其他

对于**SELU激活函数**有个“自归一化”的特性，只要保证输入权重的方差等于`1 / fan_in`，网络输出自动保持均值为0、方差为1。这就要求gain为1，但是pytorch给SELU默认的gain是3/4，目的是“牺牲归一化效果，以换取矩形层中更稳定的梯度流”

所以如果需要复刻SELU论文中的效果，请自行传入`nonlinearity='linear'`让gain为1

另外，Kaiming初始化的fan_in和fan_out默认**矩阵的转置**乘法，即x @ w.T。如果自己实现线性层需要注意，使用`nn.Linear`无所谓这个

### 损失函数&反向传播

- nn.MSELoss
- nn.CrossEntropyLoss （注意这个会对输入进行softmax，所以模型输出最后一步不需要加一个softmax）

```py
loss = nn.CrossEntropyLoss()
# Example of target with class probabilities
input = torch.randn(3, 5, requires_grad=True)
target = torch.randn(3, 5).softmax(dim=1)
output = loss(input, target) # 计算损失
output.backward() # 计算梯度
```

调用`.backward()`计算的梯度会累积，每一个batch要清空梯度

## torch.optim

`import torch.optim as optim`

要实例化优化器，传入的第一个参数是模型的所有参数`model.parameters()`

```py
# train_epoch内，每个batch更新一次
loss_fn = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.001)
for input, target in dataset:
    optimizer.zero_grad() # 清空积累的梯度
    output = model(input)
    loss = loss_fn(output, target)
    loss.backward()
    optimizer.step() # 进行一次参数更新
```

不同优化器都有自己的参数，学习率是都有的

### LRScheduler

学习率调度器 `import torch.optim.lr_scheduler`

要实例化调度器，第一个参数是优化器

```py
# main内，每个epoch步进一次
model = EasyMLP().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.AdamW(model.parameters(), lr=LEARNING_RATE, weight_decay=1e-4)
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=EPOCHS)

# 训练
train_loss, train_acc = [], []
for epoch in range(1, EPOCHS + 1):
    loss, acc = train_epoch(model, trainloader, criterion, optimizer)
    train_loss.append(loss)
    train_acc.append(acc)

    # 每个 epoch 结束后步进 scheduler
    scheduler.step()
```

常用调度器：

- 线性调度器 `LinearLR`
- 固定间隔乘系数 `StepLR` 需要定义`step_size`和系数`gamma`
- 余弦退火 `CosineAnnealingLR` 必须定义`T_max`，可定义`eta_min`（一般0或者0.1初始LR）
- 带重启余弦退火 `CosineAnnealingWarmRestarts` `T_0`初始重启周期和加倍系数`T_mult`（一般2）
- 平台衰减 `ReduceLROnPlateau` `mode`字符串min/max，衰减系数`factor`，平台期`patience`
- `OneCycleLR` 配合SGD使用，注意在batch中step

如果需要不同阶段使用不同调度器，比如warmup使用LinearLR，之后余弦退火，需要用`SequentialLR`来组合，如下：

```py
import torch
from torch.optim.lr_scheduler import LinearLR, CosineAnnealingLR, SequentialLR

# 1. 定义优化器（base_lr 设为 warmup 结束时的目标学习率）
optimizer = torch.optim.Adam(model.parameters(), lr=0.1)  

warmup_epochs = 5
total_epochs = 100

# 2. 预热调度器：从 base_lr * 0.01 线性增长到 base_lr（即 0.1）
scheduler_warmup = LinearLR(
    optimizer, 
    start_factor=0.01,   # 初始 lr = 0.1 * 0.01 = 0.001
    end_factor=1.0,      # 结束时 lr = 0.1 * 1.0 = 0.1
    total_iters=warmup_epochs
)

# 3. 余弦退火调度器：从当前 lr（0.1）衰减到 eta_min（0）
#    注意 T_max 设为剩余的轮数，这样训练结束恰好降到最低点
scheduler_cosine = CosineAnnealingLR(
    optimizer, 
    T_max=total_epochs - warmup_epochs, 
    eta_min=0
)

# 4. 顺序组合：前 5 个 epoch 用 warmup，之后切换到 cosine 并一直用到结束
scheduler = SequentialLR(
    optimizer, 
    schedulers=[scheduler_warmup, scheduler_cosine], 
    milestones=[warmup_epochs]  # 在第 5 个 epoch 结束时切换
)
```

## GPU训练

使用cuda或者cpu训练，要全部所有的张量都在同一个设备上

需要添加`.to(device)`：

- 数据X,y（原始数据集或者从loader中取出）
- 实例化的model

## 一个完整示例

对CIFAR10数据集进行分类，网络是简单的MLP，添加了两个Dropout层防止过拟合，AdamW+余弦退火

在50个Epoch之后，训练集准确率47.53%，测试集准确率48.56%

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

## 保存和加载

```py
torch.save(model,"1.pth") # 完整保存
torch.save(model.state_dict(),"2.pth") # 只保存参数，优先
# 对应1
modelA = torch.load("1.pth")
# 对应2
# 先实例化model
modelB.load_state_dict(torch.load('2.pth', weights_only=True))
```

在torchvision.models下提供了完成预训练的模型

```py
vgg16 = models.vgg16(weights="default") # default使用预训练的参数
vgg16.classifier[6] = nn.Linear(4096, 10)  # 替换
```

## tensorboard

`from torch.utils.tensorboard import SummaryWriter`

输出和打印日志用的
```py
writer = SummaryWriter("logs") # 指定路径，这里保存在logs文件夹
for i in range(100):
    writer.add_scalar("y=x",i,i)
    # 第一个参数是tag标题
    # 第二个参数是value，第三个是global_step
writer.close()
```

也可以打开图片，注意需要传入tensor/np array/str
```py
img = Image.open("D:\\dataset\\hymenoptera_data\\train\\ants\\0013035.jpg")
img_arr = np.array(img)

writer = SummaryWriter("./logs") # 相对路径
# 因为img_arr的shape是(height,width,channels)，需要匹配这个shape
writer.add_image("test",img_arr,1,dataformats="HWC")
writer.close()
```

默认保存在代码所在目录，在命令行中运行
```bash
tensorboard --logdir logs
```
就会在本地打开该日志

也可以显示网络的结构
`writer.add_graph(model,input)`