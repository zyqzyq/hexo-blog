---
title: 使用Sphinx为你的python模块自动生成文档
urlname: 0236d0a994c38bce71e1b6a299a09eb9
date: '2018-02-28 17:42:32 +0800'
layout: post
categories: python
tags:
  - python
  - sphinx
---

#### 前言

Sphinx 是一个可以用于 Python 的自动文档生成工具，可以自动的把 docstring 转换为文档，并支持多种输出格式包括 html，latex，pdf 等。

## 安装

```
pip install sphinx
```

## 生成文档的简单例子

假设现在我们有一个叫 run.py 的文件，如下

```
# run.py
def run(name):
    """
    this is how we run
    :param name name of people who runs
    """
    print name, 'is running'
```

- 新建 demo 文件夹，将 run.py 拷入
- 在 demo/ 中新建 doc 文件夹
- 生成 Sphinx 默认模板，设置项目名为 demo

```
sphinx-quickstart
```

```
注意
autodoc: automatically insert docstrings from modules (y/n) [n]选择yes
Separate source and build directories (y/n) [n]:建议选择yes
其他选项均默认即可。
生成的默认目录：
	- demo/ # 刚才新建的目录
	    - source/ # 存放Sphinx工程源码
	    - build/ # 存放生成的文档
	    Makefile
	    make.bat
```

- 打开 source/index.rst 修改为如下内容

```
Welcome to demo's API Documentation
======================================
.. toctree::
   :maxdepth: 2
   :caption: Contents:
Introduction
============
This is the introduction of demo。
API
===
.. automodule:: run
   :members:
Indices and tables
==================
* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
```

- 打开 source/conf.py 添加如下内容

```
import os
import sys
sys.path.insert(0, os.path.abspath('../..'))
```

- 在 demo 目录下运行 Sphinx 生成 html 文档

```
sphinx-build -b html source build
make html
```

- 现在，打开 build/html/index.html 就可以看到运行成功的界面了。

转载自：[http://blog.csdn.net/preyta/article/details/73647937](http://blog.csdn.net/preyta/article/details/73647937)
