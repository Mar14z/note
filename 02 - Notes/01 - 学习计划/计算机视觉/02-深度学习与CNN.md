---
title: 深度学习与 CNN
type: 阶段笔记
created: 2026-06-03
updated: 2026-10-10
tags:
  - 深度学习
  - CNN
  - PyTorch
status: 草稿
---

# 阶段 2：深度学习与 CNN

> 周期：4 周
> 目标：掌握 PyTorch 和卷积神经网络
> 状态：📋 待开始
> 前置：阶段 1

---

## 📅 周计划

### 第 5 周：PyTorch 基础
- [ ] PyTorch 安装与环境
- [ ] Tensor 基本操作
- [ ] 自动求导 (Autograd)
- [ ] DataLoader 数据加载
- [ ] GPU 加速

### 第 6 周：神经网络基础
- [ ] 感知机与多层感知机 (MLP)
- [ ] 激活函数（ReLU、Sigmoid、Tanh）
- [ ] 损失函数（CrossEntropy、MSE）
- [ ] 优化器（SGD、Adam）
- [ ] 反向传播算法

### 第 7 周：CNN 卷积神经网络
- [ ] 卷积层与池化层
- [ ] 感受野与参数量
- [ ] 经典 CNN（LeNet、AlexNet）
- [ ] 特征图可视化
- [ ] 梯度问题（消失/爆炸）

### 第 8 周：训练技巧与实战
- [ ] 数据增强（Data Augmentation）
- [ ] 批归一化（BatchNorm）
- [ ] Dropout 与正则化
- [ ] 学习率调度
- [ ] 实战项目：CIFAR-10 分类

---

## 📖 核心知识点

### 1. PyTorch 基础

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader

# Tensor 操作
x = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)
y = x.cuda()  # 移动到 GPU
z = torch.matmul(x, y)  # 矩阵乘法

# 自动求导
x = torch.randn(3, requires_grad=True)
y = x ** 2
y.sum().backward()
print(x.grad)

# 模型定义
class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(784, 128)
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return self.fc2(x)
```

### 2. CNN 核心组件

```python
# 卷积层
nn.Conv2d(in_channels, out_channels, kernel_size, stride, padding)

# 池化层
nn.MaxPool2d(kernel_size, stride)
nn.AvgPool2d(kernel_size, stride)

# 批归一化
nn.BatchNorm2d(num_features)

# Dropout
nn.Dropout(p=0.5)
```

### 3. 经典 CNN 模型

```python
# LeNet-5 (1998)
class LeNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16*5*5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

# AlexNet (2012)
class AlexNet(nn.Module):
    def __init__(self, num_classes=1000):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 11, 4, 2),
            nn.ReLU(),
            nn.MaxPool2d(3, 2),
            nn.Conv2d(64, 192, 5, padding=2),
            nn.ReLU(),
            nn.MaxPool2d(3, 2),
            nn.Conv2d(192, 384, 3, padding=1),
            nn.ReLU(),
            nn.Conv2d(384, 256, 3, padding=1),
            nn.ReLU(),
            nn.Conv2d(256, 256, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(3, 2),
        )
        self.classifier = nn.Sequential(
            nn.Linear(256*6*6, 4096),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(4096, num_classes),
        )
```

### 4. 训练循环

```python
def train(model, dataloader, criterion, optimizer, device):
    model.train()
    total_loss = 0
    for inputs, labels in dataloader:
        inputs, labels = inputs.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    return total_loss / len(dataloader)

def evaluate(model, dataloader, device):
    model.eval()
    correct = 0
    total = 0
    with torch.no_grad():
        for inputs, labels in dataloader:
            inputs, labels = inputs.to(device), labels.to(device)
            outputs = model(inputs)
            _, predicted = outputs.max(1)
            total += labels.size(0)
            correct += predicted.eq(labels).sum().item()
    return correct / total
```

### 5. 数据增强

```python
from torchvision import transforms

train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                       std=[0.229, 0.224, 0.225])
])
```

---

## 🎯 实战项目

### 项目：CIFAR-10 图像分类

**数据集**：
- 60,000 张 32x32 彩色图片
- 10 个类别（飞机、汽车、鸟、猫、鹿、狗、青蛙、马、船、卡车）
- 50,000 训练 + 10,000 测试

**目标**：
- 准确率达到 **85%+**
- 理解完整训练流程
- 学习超参数调优

**技术栈**：
- PyTorch
- ResNet-18（迁移学习）
- 数据增强
- 学习率调度
- TensorBoard 可视化

**实施步骤**：
1. 数据加载与增强
2. 模型定义（ResNet-18）
3. 训练与验证循环
4. 测试集评估
5. 混淆矩阵分析
6. 模型保存

---

## 📚 学习资源

### 官方文档
- PyTorch 官方教程: https://pytorch.org/tutorials/
- PyTorch 中文文档: https://pytorch.apachecn.org/

### 课程
- 李沐《动手学深度学习》https://zh.d2l.ai/
- 吴恩达深度学习课程
- Fast.ai 实战课程

### 书籍
- 《深度学习》（花书）
- 《Python 深度学习》（François Chollet）

---

## ✅ 阶段考核

- [ ] 独立搭建 CNN 模型
- [ ] 在 CIFAR-10 上达到 85%+ 准确率
- [ ] 理解数据增强的重要性
- [ ] 掌握模型保存与加载
- [ ] 写一篇学习总结

---

## 💡 关键概念速记

| 概念 | 一句话解释 |
|------|----------|
| 卷积 | 滑动窗口提取特征 |
| 池化 | 降采样，减少计算量 |
| 感受野 | 输出像素对应输入的区域 |
| 批归一化 | 归一化每批数据，加速收敛 |
| Dropout | 随机丢弃神经元，防止过拟合 |
| 迁移学习 | 用预训练模型微调新任务 |
| 学习率 | 控制参数更新步长 |
| 损失函数 | 衡量预测与真实值的差距 |

---

*深度学习是现代 CV 的核心，必须深入理解！*
