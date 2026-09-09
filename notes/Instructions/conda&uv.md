---
title: "Python虚拟环境"
date: 2026-07-18
tags: [编程]
---

# conda

cls 清屏

## 虚拟环境

conda env list //列出当前所有的虚拟环境
conda create -n <环境名> python=3.10.2 //创建虚拟环境，默认在conda的安装路径下
conda remove -n <环境名> --all //删

## 环境内

conda activate <环境名>
conda list //列出当前环境下的所有库
conda deactivate

pip show <库名> //查看某个库的版本
pip install <库名>==<版本号> (-i <URL>)

// pytorch不能直接这样安装，去官网匹配自己GPU的版本，需要自己GPU的CUDA版本 大于等于 pytorch的cuda版本；也要匹配python版本

# uv

使用

```
.venv\Scripts\activate
```

来激活虚拟环境

## 安装后配置

修改环境变量，把uv下载的包缓存在D盘

```powershell
setx UV_CACHE_DIR D:\uv-cache
```

（否则默认在C盘，如果项目跨盘的话，uv的hardlink机制会失败，会复制一份一模一样的包在项目的`.venv`下）

## 虚拟环境

### init

init会配置git和readme等文件，在当前目录创建一个my-project文件夹

uv init my-project -p 3.12

### venv

只创建环境

uv venv my-env -p 3.11      # 指定环境名字和python版本

## 包管理

uv add # 为项目加包
uv pip install # 添加一些临时的包的时候，不会记录在pyproject

uv sync # 如果有一个uv的项目，直接sync即可配置好环境（如果需要复现旧环境 加上 --frozen）

uv pip freeze > requirements.txt

## pyproject.toml

配置清华源
```
[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

下载torch时注意，默认只会下载cpu版本，需要从官网找到版本，手动添加
```
[[tool.uv.index]]
name = "pytorch-cu132"
url = "https://download.pytorch.org/whl/cu132"
explicit = true

[tool.uv.sources]
torch = { index = "pytorch-cu132" }
torchvision = { index = "pytorch-cu132" }
torchaudio = { index = "pytorch-cu132" }
```

# 其他

```python
import torch

if torch.cuda.is_available():
    print(f"PyTorch version: {torch.__version__}")
    print(f"CUDA version: {torch.version.cuda}")
    print(f"GPU count: {torch.cuda.device_count()}")
    print(f"Current GPU: {torch.cuda.get_device_name(0)}")
else:
    print("No CUDA device detected.")
```