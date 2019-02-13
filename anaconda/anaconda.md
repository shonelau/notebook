# anaconda 笔记
[TOC]
## 查看基本信息
```bash
# 查看anaconda 基本信息
conda info 
conda --version
# 查看anaconda 已安装的python环境
conda env list
# or 
conda info --env
```


## 使用默认base环境
```bash
# 激活默认python
conda activate
# 退出当前环境 
conda deactivate
```
## 安装新环境

```bash
# 新增python2.7环境，取名为python27,版本为2.7
conda create -n python27 python=2.7
# 使用新增的python2.7
conda activate python27
# 在当前环境下安装第三方包
conda install pip
#### conda install numpy
conda install panda
...
#或者
pip install numpy

# 在当前环境下删除第三方包
conda remove panda
#或者
pip uninstall panda
# 查看当前环境下安装的包
conda list

# 转到anaconda base
conda activate

# 再转回python2.7
conda deactivate

# 由于可以不同的环境间转移，通过conda deactivate 回退到上个环境，直到完全退出。

```

## 环境管理
```bash
# 删除一个环境,及其所有包
conda remove -n python27 --all

# 导出当前环境的包信息
conda env export > environment.yaml 
# 用配置文件创建新的虚拟环境
conda env create -f environment.yaml 
```

