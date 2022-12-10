---
title: 2018-02-28-使用Sphinx为你的python模块自动生成文档
urlname: 0236d0a994c38bce71e1b6a299a09eb9
date: '2022-12-09 20:02:21 +0800'
tags: []
categories: []
---

<p>---

author: zyqzyq

date: 2018-02-28 17:42:32+00:00

layout: post

title: 使用 Sphinx 为你的 python 模块自动生成文档

categories: python

key: 20180228

tags:</p>

<ul>
<li>python</li>
<li>sphinx</li>
</ul>
<hr />
<h4>前言</h4>
<p>Sphinx是一个可以用于Python的自动文档生成工具，可以自动的把docstring转换为文档，并支持多种输出格式包括html，latex，pdf等。</p>
<h2>安装</h2>
<pre><code>pip install sphinx
</code></pre>
<h2>生成文档的简单例子</h2>
<p>假设现在我们有一个叫run.py的文件，如下</p>
<pre><code># run.py
def run(name):
    """
    this is how we run

    :param name name of people who runs
    """
    print name, 'is running'

</code></pre>

<ul>
<li>新建demo 文件夹，将run.py 拷入</li>
<li>在demo/ 中新建doc文件夹</li>
<li>生成Sphinx默认模板，设置项目名为demo</li>
</ul>
<pre><code>sphinx-quickstart 
</code></pre>
<pre><code>注意
autodoc: automatically insert docstrings from modules (y/n) [n]选择yes
Separate source and build directories (y/n) [n]:建议选择yes
其他选项均默认即可。
生成的默认目录：
	- demo/ # 刚才新建的目录
	    - source/ # 存放Sphinx工程源码
	    - build/ # 存放生成的文档
	    Makefile
	    make.bat
</code></pre>
<ul>
<li>打开source/index.rst 修改为如下内容</li>
</ul>
<pre><code>Welcome to demo's API Documentation
======================================

.. toctree::
:maxdepth: 2
:caption: Contents:

# Introduction

This is the introduction of demo。

# API

.. automodule:: run
:members:

# Indices and tables

- :ref:`genindex`
- :ref:`modindex`
- :ref:`search`
</code></pre>
<ul>
<li>打开source/conf.py 添加如下内容</li>
</ul>
<pre><code>import os
import sys
sys.path.insert(0, os.path.abspath('../..'))
</code></pre>
<ul>
<li>在demo目录下运行Sphinx生成html文档</li>
</ul>
<pre><code>sphinx-build -b html source build
make html
</code></pre>
<ul>
<li>现在，打开build/html/index.html就可以看到运行成功的界面了。</li>
</ul>
<p>转载自：<a href="http://blog.csdn.net/preyta/article/details/73647937" target="_blank">http://blog.csdn.net/preyta/article/details/73647937</a></p>
