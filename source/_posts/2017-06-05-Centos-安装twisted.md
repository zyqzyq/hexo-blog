---
title: 2017-06-05-Centos-安装twisted
urlname: f9d923c18b4170b6b1cf8a2caca9358d
date: '2022-12-09 20:02:11 +0800'
tags: []
categories: []
---

<hr />
<p>layout: post

title: "Centos 安装 twisted"

date: 2017-06-05 16:25:06

categories: Centos

key: 20170605

tags:</p>

<ul>
<li>centos</li>
<li>twisted</li>
</ul>
<hr />
<p>请尊重分享成果，转载请注明出处，本文来自Alex_Xia，Building_Home,原文链接：

<a href="http:_www.cnblogs.com_alex-xia_p_6062741" target="_blank">http://www.cnblogs.com/alex-xia/p/6062741.html</a>

<a href="http:_www.centoscn.com_image-text_install_2015_0129_4584" target="_blank">http://www.centoscn.com/image-text/install/2015/0129/4584.html</a></p>

<h2>环境：</h2>
<p>腾讯云Centos 7.2 64位

Python 2.7.12 64 位</p>

<hr />
<h2>一、更新python</h2>
<pre><code>目前CentOS7.2自带的python版本是python2.7.5。由于yum这个软件需要系统自带的python工作。
如果冒然用自己安装的python替换掉系统自带的，可能造成yum不工作。
网上教程提示先安装 readline-devel ，openssl-devel安装包，我安装后发现电脑自带了，如未安装可以运行以下命令安装
</code></pre>
<pre><code>yum install -y readline-devel
yum install -y openssl-devel
</code></pre>
<p>下载python并解压</p>
<pre><code>cd /tmp
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
tar -xJvf Python-2.7.12.tar.xz
</code></pre>
<p>编译</p>
<pre><code>cd Python-2.7.12/
./configure --prefix=/usr/local/python2.7
make
make install
</code></pre>
<p>链接</p>
<pre><code>ln -s /usr/local/python2.7/bin/python2.7 /usr/local/bin/python
</code></pre>
<p>由于系统自带的python路径是/usr/bin/python。PATH中，/usr/local/bin比/usr/bin靠前，所以当你输入python，系统会自动启动你安装的python2.7.12。</p>
<pre><code>echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
</code></pre>
<h2>二、安装easy_install,pip</h2>
<p>CentOS 安装easy_install的方法：</p>
<pre><code>wget -q http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
</code></pre>
<p>CentOS安装python包管理安装工具pip的方法如下：

注意：wget 获取 https 的时候要加上：--no-check-certificate</p>

<pre><code>wget --no-check-certificate https://github.com/pypa/pip/archive/1.5.5.tar.gz
tar zvxf 1.5.5.tar.gz    #解压文件
cd pip-1.5.5/
python setup.py install
</code></pre>
<h2>三、安装twisted</h2>
<pre><code>pip install twisted 
</code></pre>
<p>报错：no module named bz2

解决办法：</p>

<pre><code>yum install bzip2*
</code></pre>
<p>然后重新编译python,重新安装easy_install,pip。

测试：</p>

<pre><code>Python 2.7.12 (default, Jun  4 2017, 11:19:07)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-11)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import twisted
>>>

</code></pre>
<p>成功。</p>
