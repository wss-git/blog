---
title: 使用 conda 管理包
categories:
  - 笔记
excerpt: 'conda 的简单实用'
description: 'conda 的简单实用'
date: 2024-01-09 10:17:19
---


## 安装

参考[官网](https://docs.conda.io/en/latest/miniconda.html%EF%BC%89%EF%BC%8C%E9%80%89%E6%8B%A9%E9%80%82%E5%90%88%E4%BD%A0%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%89%88%E6%9C%AC%EF%BC%8C%E7%84%B6%E5%90%8E%E4%B8%8B%E8%BD%BD%E3%80%82)

## 常用命令

1. 创建一个新的环境：
```
conda create --name env_name
```
其中，`env_name`是你想要创建的环境的名称。

2. 激活/进入一个环境：
```
conda activate env_name
```
其中，`env_name`是你想要激活的环境的名称。

3. 退出当前环境：
```
conda deactivate
```

4. 查看已安装的所有环境：
```
conda env list
```

5. 删除一个环境：
```
conda env remove --name env_name
```
其中，`env_name`是要删除的环境的名称。

6. 安装包：
```
conda install package_name
```
其中，`package_name`是要安装的包的名称。

7. 更新包：
```
conda update package_name
```
其中，`package_name`是要更新的包的名称。

8. 删除包：
```
conda remove package_name
```
其中，`package_name`是要删除的包的名称。

9. 查找可用的包：
```
conda search package_name
```
其中，`package_name`是要查找的包的名称。

10. 导出/备份环境配置：
```
conda env export > environment.yml
```

11. 从环境配置文件中创建一个新的环境：
```
conda env create -f environment.yml
```

这些命令可以帮助你更好地使用`conda`。更多关于`conda`的使用方法，请参考[官方文档](https://docs.conda.io/en/latest/)