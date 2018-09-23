---
layout: post
title: Windows环境下如何设置pip2和pip3共存
categories: Python
description: hhhhhhhhhhhhhh
keywords:
---

## 1 安装Python2和Python3
下载Python的安装包，安装好Python2和Python3。

## 2 配置环境变量
配置好Python3的环境变量C:\ProgramData\Anaconda3;C:\ProgramData\Anaconda3\Scripts，至此已经可以在cmd里面运行python和pip命令了。
进入Python3的安装目录，将python.exe和pythonw.exe，更改为python3.exe和pythonw3.exe，之后，就在cmd的命令行里输入：
```python
python3 -m pip install --upgrade pip --force-reinstall
```
至此python3和pip3命令已经可以在cmd里面使用了。

至此python2和pip2命令以此类推。
