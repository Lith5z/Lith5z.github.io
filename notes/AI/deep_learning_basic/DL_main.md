---
title: "笔记 深度学习"
date: 2026-09-14
tags: [AI, DL]
---

# 深度学习DL 概述

## 流程

```
划分样本 -> (数据增强) -> 读入数据 -> 前向传播 -> 计算损失 -> 反向传播 -> 更新参数
                                       ^                                 |
                                       |                                 |
                                       -----------------------------------
```

## 样本

- 训练集：训练模型
- （验证集：调整超参数）
- 测试集：模拟真实世界，评估模型泛化能力

使用`sklearn`库
```py
from sklearn.model_selection import train_test_split

# 定义特征 X 和目标标签 y (Survived)
X = df.drop('Survived', axis=1)
y = df['Survived']

# 划分训练集和测试集（80% 训练，20% 测试）
X_train，X_test，y_train，y_test = train_test_split(
    X，y,
    test_size=0.2,
    random_state=42, # 固定随机种子，保证可复现
    stratify=y # 分层抽样，保持正负样本比例一致
)

# 之后所有的预处理只能在 X_train 测试集进行
```

## 数据增强/预处理

现实数据存在的问题：

- 存在缺失值和异常值 -> 数据清洗
- 包含文本、类别等非数值类型 -> 特征编码
- 量纲差异巨大（房价是百万级，房间数是个位数） -> 特征缩放

具体操作见[[数据预处理]]

### 数据清洗

对于有大量数据缺失（稀疏）的特征，可以考虑直接删除这一维度，或者进行填充

对于过大/小的**异常值**，考虑是否进行截断

### 特征编码

对于非数值的特征，如果反映了程度的大小（比如学历/满意度），可以按着大小进行序数编码；如果是无序的（颜色/血型），提供独热编码 One-Hot，把每个类别变成独立的列

### 特征缩放

- 对于符合正态分布的数值（身高/体重），进行标准化，得到均值为0、标准差为1的分布
- 对于天然有界的数据（像素的RGB信息/Sigmoid函数输出），进行归一化

## 参数初始化

[[参数初始化]]

- Xavier初始化：sigmoid/tanh
- Kaiming初始化：ReLU等

## 前向传播

输入数据，逐层计算得到预测值
通过预测值和真实值，带入损失函数计算损失

**损失函数**

[[损失函数]]

- 均方差 Mean Squared Error/MSE $\frac{1}{n} \sum (\hat y - y)^2$
- 交叉熵 [[似然估计]]

**激活函数**

[[激活函数]]

- Sigmoid
- Softmax
- ReLU
- GeLU
- SiLU

区分：

- epoch：轮次，对全部测试集的一次完整训练，一般分多个batch
- batch：批次，每个epoch之内读入部分测试集，进行训练

### 归一化

[[归一化]]

- batch norm
- layer norm
- group norm
- RMSnorm

## 反向传播

我们希望预测值接近真实值，就要最小化损失函数，对损失函数求每个参数的梯度
反向传播提供了为多层神经网络计算每个参数的梯度的方法

- [[3B1B 反向传播算法]]

## 更新参数

优化器：通过反向传播得到梯度之后，如何调整参数

**梯度下降/一阶优化算法**
$$ \text{GD:}\quad \theta \leftarrow \theta - \eta \nabla_{\theta} L(\theta) $$

$\theta$ 是模型的所有参数，$\eta$ 是学习率/learning_rate，$L(x)$ 是损失函数

具体可见[[优化器-梯度下降和动量]]、[[优化器-自适应学习率]]

根据梯度下降得到的不同优化器：

- BGD：根据所有预测数据计算梯度并更新参数，精确但开销大，不用
- SGD：选择一条预测数据计算并更新
- Mini-batch SGD：选择一小批的预测数据计算并更新，常用
- Momentum 动量法：参数更新时积累历史的速度，适用于CNN等经典图像模型
- AdaGrad 自适应梯度：每个参数维护自己的学习率，累积历史梯度平方和来减小学习率，适用于稀疏数据场景，比如自然语言处理中的词向量训练、推荐系统中的大规模稀疏特征学习
- RMSProp：用指数滑动平均代替平方和积累，从而更关注最近的梯度变化，适合非平稳目标函数的优化，比如循环神经网络RNN的训练
- Adam/AdamW：RMSProp+动量

> 注：现代的优化器只要不指明是全批量的梯度下降，就都是跟SGD一样分batch更新参数

**二阶优化算法**

- 牛顿法
- BFGS/L-BFGS

[[学习率调度]]：

- LinearLR 线性调度器
- StepLR 固定间隔乘系数
- CosineAnnealingLR 余弦退火
- ReduceLROnPlateau 平台衰减
- OneCycleLR

## 模型评估

R方
欠拟合/过拟合
[[正则化]]

[谦行 模型评估](https://www.yuque.com/qx2io/smtwex/vcqzedxf0slg89sq)

## 一般代码框架

可以结合pytorch的笔记看

```py
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader, random_split
import numpy as np
import os
from tqdm import tqdm

# ==================== 1. 固定随机种子 ====================
def set_seed(seed=42):
    """保证结果可复现"""
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
        torch.backends.cudnn.deterministic = True
        torch.backends.cudnn.benchmark = False

# ==================== 2. 数据集定义 ====================
class MyDataset(Dataset):
    """自定义数据集，适用于非标准格式的数据"""
    
    def __init__(self, data_dir, transform=None, split='train'):
        """
        Args:
            data_dir: 数据根目录
            transform: 数据预处理/增强
            split: 'train', 'val', 或 'test'
        """
        self.data_dir = data_dir
        self.transform = transform
        self.split = split
        
        # TODO: 加载数据路径和标签
        # 例如：读取图片文件列表和对应标签
        self.images = []    # 图片路径列表
        self.labels = []    # 标签列表
        
        # 示例：假设数据按文件夹组织
        class_dirs = ['cat', 'dog', ...]
        for class_idx, class_name in enumerate(class_dirs):
            class_path = os.path.join(data_dir, class_name)
            for img_name in os.listdir(class_path):
                self.images.append(os.path.join(class_path, img_name))
                self.labels.append(class_idx)
    
    def __len__(self):
        return len(self.images)
    
    def __getitem__(self, idx):
        # 加载单个样本
        img_path = self.images[idx]
        label = self.labels[idx]
        
        # 加载图片（示例，实际根据数据类型调整）
        # image = Image.open(img_path).convert('RGB')
        
        if self.transform:
            # image = self.transform(image)
            pass
        
        return image, label

# ==================== 3. 数据预处理/增强 ====================
def get_transforms():
    """定义训练和验证的不同变换"""
    
    # 训练集变换（包含数据增强）
    train_transform = transforms.Compose([
        transforms.RandomResizedCrop(224),
        transforms.RandomHorizontalFlip(),
        transforms.ColorJitter(brightness=0.2, contrast=0.2),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406],
                           std=[0.229, 0.224, 0.225])
    ])
    
    # 验证/测试集变换（仅标准化，无增强）
    val_transform = transforms.Compose([
        transforms.Resize(256),
        transforms.CenterCrop(224),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406],
                           std=[0.229, 0.224, 0.225])
    ])
    
    return train_transform, val_transform

# ==================== 4. 数据加载 ====================
def create_dataloaders(data_dir, batch_size=64, val_ratio=0.2, num_workers=4):
    """
    创建训练、验证、测试 DataLoader
    
    Args:
        data_dir: 数据根目录
        batch_size: 批次大小
        val_ratio: 验证集比例（从训练集划分）
        num_workers: 数据加载线程数
    
    Returns:
        train_loader, val_loader, test_loader
    """
    train_transform, val_transform = get_transforms()
    
    # 加载完整数据集
    full_dataset = MyDataset(data_dir, transform=train_transform, split='train')
    
    # 划分训练集和验证集
    train_size = int((1 - val_ratio) * len(full_dataset))
    val_size = len(full_dataset) - train_size
    train_dataset, val_dataset = random_split(
        full_dataset, 
        [train_size, val_size],
        generator=torch.Generator().manual_seed(42)  # 固定划分结果
    )
    
    # 为验证集单独设置 transform（不需要数据增强）
    val_dataset.dataset.transform = val_transform
    
    # 测试集（独立的数据集）
    test_dataset = MyDataset(os.path.join(data_dir, 'test'), 
                            transform=val_transform, 
                            split='test')
    
    # 创建 DataLoader
    train_loader = DataLoader(
        train_dataset, 
        batch_size=batch_size, 
        shuffle=True,           # 训练集打乱
        num_workers=num_workers,
        pin_memory=True,        # 加速数据传输到GPU
        drop_last=True          # 丢弃最后一个不完整的batch
    )
    
    val_loader = DataLoader(
        val_dataset, 
        batch_size=batch_size, 
        shuffle=False,          # 验证集不打乱
        num_workers=num_workers,
        pin_memory=True
    )
    
    test_loader = DataLoader(
        test_dataset, 
        batch_size=batch_size, 
        shuffle=False,
        num_workers=num_workers,
        pin_memory=True
    )
    
    return train_loader, val_loader, test_loader

# ==================== 5. 模型定义 ====================
class MyModel(nn.Module):
    """你的神经网络模型"""
    
    def __init__(self, num_classes=10):
        super(MyModel, self).__init__()
        # TODO: 定义网络层
        self.conv1 = nn.Conv2d(3, 64, 3, padding=1)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU()
        self.pool = nn.MaxPool2d(2, 2)
        # ... 更多层
        self.fc = nn.Linear(64 * 16 * 16, num_classes)
    
    def forward(self, x):
        # TODO: 前向传播
        x = self.pool(self.relu(self.bn1(self.conv1(x))))
        # ...
        x = x.view(x.size(0), -1)
        x = self.fc(x)
        return x

# ==================== 6. 训练配置 ====================
def get_optimizer_and_scheduler(model, config):
    """配置优化器和学习率调度器"""
    
    # 优化器
    if config['optimizer'] == 'sgd':
        optimizer = optim.SGD(
            model.parameters(),
            lr=config['lr'],
            momentum=config.get('momentum', 0.9),
            weight_decay=config.get('weight_decay', 5e-4)
        )
    elif config['optimizer'] == 'adam':
        optimizer = optim.Adam(
            model.parameters(),
            lr=config['lr'],
            weight_decay=config.get('weight_decay', 0)
        )
    elif config['optimizer'] == 'adamw':
        optimizer = optim.AdamW(
            model.parameters(),
            lr=config['lr'],
            weight_decay=config.get('weight_decay', 0.01)
        )
    
    # 学习率调度器
    scheduler = None
    if config['scheduler'] == 'step':
        scheduler = optim.lr_scheduler.StepLR(
            optimizer, 
            step_size=config.get('step_size', 30), 
            gamma=config.get('gamma', 0.1)
        )
    elif config['scheduler'] == 'cosine':
        scheduler = optim.lr_scheduler.CosineAnnealingLR(
            optimizer, 
            T_max=config['epochs']
        )
    elif config['scheduler'] == 'reduce':
        scheduler = optim.lr_scheduler.ReduceLROnPlateau(
            optimizer, 
            mode='min', 
            factor=0.5, 
            patience=5
        )
    
    return optimizer, scheduler

# ==================== 7. 训练一个epoch ====================
def train_one_epoch(model, train_loader, criterion, optimizer, device, epoch):
    """训练一个epoch的核心逻辑"""
    
    model.train()  # 设置为训练模式
    running_loss = 0.0
    correct = 0
    total = 0
    
    # 使用进度条
    pbar = tqdm(train_loader, desc=f'Epoch {epoch}')
    
    for batch_idx, (inputs, targets) in enumerate(pbar):
        # 1. 数据移到设备（GPU/CPU）
        inputs, targets = inputs.to(device), targets.to(device)
        
        # 2. 清零梯度（重要！）
        optimizer.zero_grad()
        
        # 3. 前向传播
        outputs = model(inputs)
        
        # 4. 计算损失
        loss = criterion(outputs, targets)
        
        # 5. 反向传播（计算梯度）
        loss.backward()
        
        # 6. 梯度下降（更新参数）
        optimizer.step()
        
        # 7. 统计信息
        running_loss += loss.item()
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()
        
        # 更新进度条显示
        pbar.set_postfix({
            'Loss': f'{loss.item():.4f}',
            'Acc': f'{100.*correct/total:.2f}%'
        })
    
    epoch_loss = running_loss / len(train_loader)
    epoch_acc = 100. * correct / total
    
    return epoch_loss, epoch_acc

# ==================== 8. 验证/评估 ====================
@torch.no_grad()  # 装饰器：不计算梯度，节省内存
def evaluate(model, val_loader, criterion, device):
    """验证/测试，不更新参数"""
    
    model.eval()  # 设置为评估模式（影响Dropout、BN等）
    
    running_loss = 0.0
    correct = 0
    total = 0
    
    for inputs, targets in val_loader:
        inputs, targets = inputs.to(device), targets.to(device)
        
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        
        running_loss += loss.item()
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()
    
    val_loss = running_loss / len(val_loader)
    val_acc = 100. * correct / total
    
    return val_loss, val_acc

# ==================== 9. 保存和加载模型 ====================
def save_checkpoint(model, optimizer, epoch, best_acc, filepath):
    """保存完整训练状态（用于恢复训练）"""
    checkpoint = {
        'epoch': epoch,
        'model_state_dict': model.state_dict(),
        'optimizer_state_dict': optimizer.state_dict(),
        'best_acc': best_acc,
    }
    torch.save(checkpoint, filepath)
    print(f'Checkpoint saved to {filepath}')

def load_checkpoint(filepath, model, optimizer=None):
    """加载训练状态"""
    checkpoint = torch.load(filepath)
    model.load_state_dict(checkpoint['model_state_dict'])
    
    if optimizer:
        optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
    
    epoch = checkpoint.get('epoch', 0)
    best_acc = checkpoint.get('best_acc', 0)
    
    return model, optimizer, epoch, best_acc

def save_best_model(model, acc, filepath='best_model.pth'):
    """只保存最佳模型参数"""
    torch.save(model.state_dict(), filepath)
    print(f'Best model saved with acc: {acc:.2f}%')

# ==================== 10. 主训练流程 ====================
def main():
    # 配置参数
    config = {
        'epochs': 100,
        'batch_size': 128,
        'lr': 0.1,
        'optimizer': 'sgd',      # 'sgd', 'adam', 'adamw'
        'scheduler': 'cosine',   # 'step', 'cosine', 'reduce', None
        'momentum': 0.9,
        'weight_decay': 5e-4,
        'step_size': 30,
        'gamma': 0.1,
        'val_ratio': 0.2,
        'num_workers': 4,
        'device': 'cuda' if torch.cuda.is_available() else 'cpu',
        'data_dir': './data',
        'num_classes': 10,
        'seed': 42
    }
    
    # 1. 固定随机种子
    set_seed(config['seed'])
    
    # 2. 设置设备
    device = torch.device(config['device'])
    print(f'Using device: {device}')
    
    # 3. 加载数据
    train_loader, val_loader, test_loader = create_dataloaders(
        data_dir=config['data_dir'],
        batch_size=config['batch_size'],
        val_ratio=config['val_ratio'],
        num_workers=config['num_workers']
    )
    print(f'Train: {len(train_loader.dataset)} | Val: {len(val_loader.dataset)} | Test: {len(test_loader.dataset)}')
    
    # 4. 创建模型
    model = MyModel(num_classes=config['num_classes']).to(device)
    print(f'Total parameters: {sum(p.numel() for p in model.parameters()):,}')
    
    # 5. 损失函数
    criterion = nn.CrossEntropyLoss()
    
    # 6. 优化器和调度器
    optimizer, scheduler = get_optimizer_and_scheduler(model, config)
    
    # 7. 训练记录
    best_val_acc = 0.0
    train_losses, train_accs = [], []
    val_losses, val_accs = [], []
    
    # 8. 训练循环
    print('\n' + '='*60)
    print('Starting Training...')
    print('='*60)
    
    for epoch in range(1, config['epochs'] + 1):
        # 训练一个epoch
        train_loss, train_acc = train_one_epoch(
            model, train_loader, criterion, optimizer, device, epoch
        )
        
        # 验证
        val_loss, val_acc = evaluate(
            model, val_loader, criterion, device
        )
        
        # 记录
        train_losses.append(train_loss)
        train_accs.append(train_acc)
        val_losses.append(val_loss)
        val_accs.append(val_acc)
        
        # 更新学习率（注意：ReduceLROnPlateau需要传入验证损失）
        if scheduler:
            if config['scheduler'] == 'reduce':
                scheduler.step(val_loss)
            else:
                scheduler.step()
        
        # 保存最佳模型
        if val_acc > best_val_acc:
            best_val_acc = val_acc
            save_best_model(model, best_val_acc)
        
        # 打印进度
        current_lr = optimizer.param_groups[0]['lr']
        print(f'Epoch [{epoch:3d}/{config["epochs"]}] | '
              f'Train Loss: {train_loss:.4f} | Train Acc: {train_acc:.2f}% | '
              f'Val Loss: {val_loss:.4f} | Val Acc: {val_acc:.2f}% | '
              f'LR: {current_lr:.6f}')
    
    # 9. 最终测试
    print('\n' + '='*60)
    print('Final Evaluation on Test Set')
    print('='*60)
    
    # 加载最佳模型
    model.load_state_dict(torch.load('best_model.pth'))
    
    test_loss, test_acc = evaluate(model, test_loader, criterion, device)
    print(f'Test Loss: {test_loss:.4f} | Test Accuracy: {test_acc:.2f}%')
    
    # 10. 可选：绘制训练曲线
    # plot_curves(train_losses, val_losses, train_accs, val_accs, best_val_acc)

if __name__ == '__main__':
    main()
```