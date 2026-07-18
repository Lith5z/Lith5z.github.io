---
title: "Python虚拟环境"
date: 2026-07-18
tags: [编程]
---

# Conda 命令行

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

//pytorch不能直接这样安装，去官网匹配自己GPU的版本，需要自己GPU的CUDA版本 大于等于 pytorch的cuda版本；也要匹配python版本

# uv

## 虚拟环境

uv venv my-env -p 3.11      # 指定环境名字和python版本
uv venv -p python@3.11 # 使用 python3.11 创建 .venv

uv init # 初始化uv项目
uv add # 为项目加包
uv sync # 如果有拉取一个uv的项目，直接sync即可配置好环境
uv pip freeze > requirements.txt # 生成一个包含所有已安装包及其精确版本的列表，格式与 pip freeze 的输出相同，常用于生成 requirements.txt 文件

## pyproject.toml

配置清华源
```
[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

下载torch时注意，默认只会下载cpu版本，手动添加
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