---
layout: post
title:  "Anoconda常用命令入门"
date:   2019-05-10 07:21:20 +0800
categories: Python
tag: Python
---

Python 中的 Anaconda 等价于 Ruby中的rvm, 由于我之前从事过Ruby开发, 对rvm的使用非常熟悉, 所以理解起Anaconda很容易, 须要了解Anaconda更多概念, 推荐这篇 <a href="https://www.zhihu.com/question/58033789" target="_blank">详细介绍下Anaconda</a>

### 一、如何安装Anaconda?

官网下载: continuum.io

### 二、更新Anaconda里所有包

conda upgrade --all

### 三、创建环境并指定要安装在环境中的Python版本

// conda create -n env_name package_names
conda create -n py3 python=3

如果你要安装特定版本（例如 Python 3.6），请使用 conda create -n py python=3.6

### 四、进入环境

使用 source activate my_env 进入环境

### 五、离开环境

conda deactivate

### 六、列出已安装的包

conda list

### 七、列出环境

conda env list

### 八、删除环境

conda env remove -n env_name

### 九、共享环境

你可以在你当前的环境中终端中使用 
conda env export > environment.yaml

// 其中-f表示你要导出文件在本地的路径，所以/path/to/environment.yml要换成你本地的实际路径
conda env update -f=/path/to/environment.yml
