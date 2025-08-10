---
layout: post
title: miniconda安装及使用
slug: miniconda-installation-and-usage
categories: [环境配置]
tags: [Python]
---

## miniconda安装


```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

bash ./Miniconda3-latest-Linux-x86_64.sh
```

## miniconda替换国内源
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/
conda config --set show_channel_urls yes
```

## miniconda配置
配置文件路径在：`~/.condarc`。

可通过``conda config --show`列出所有配置项。

可通过`conda config --set <KEY> <VALUE>`设置配置项。

```bash
conda config --set auto_activate_base false
```

## conda常用命令
环境管理：
```bash
# 创建新的环境，使用默认python
conda create -n <env-name>

# 创建新的环境，使用指定python
conda create -n <env-name> python=<python-ver>

# clone已有环境
conda create -n <clone-env> --clone <existed-env>

# 激活已存在的环境
conda activate <env-name>

# 退出已激活的环境
conda deactivate

# 删除虚拟环境及其中的所有内容。
conda remove --name <env-name> --all

# 列出已有的环境
conda env list
```

包管理：
```bash
常用命令
# 查看该环境的安装的包
conda list

# 安装包
conda install <pkg-name>

# 升级安装包
conda update <pkg-name>

# 卸载指定安装包
conda uninstall <pkg-name>

# 清理不再使用的包
conda clean --packages

 #清理缓存
conda clean --all
```

## 常用conda包安装
```bash
conda install numpy pandas matplotlib plotly ipykernel notebook
```