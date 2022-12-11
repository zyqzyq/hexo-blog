---
title: Centos 安装twisted
urlname: f9d923c18b4170b6b1cf8a2caca9358d
date: '2017-06-05 16:25:06 +0800'
layout: post
categories: Centos
tags:
  - centos
  - twisted
---

请尊重分享成果，转载请注明出处，本文来自 Alex_Xia，Building_Home,原文链接：
[http://www.cnblogs.com/alex-xia/p/6062741.html](http:_www.cnblogs.com_alex-xia_p_6062741)
[http://www.centoscn.com/image-text/install/2015/0129/4584.html](http:_www.centoscn.com_image-text_install_2015_0129_4584)

## 环境：

腾讯云 Centos 7.2 64 位
Python 2.7.12 64 位

---

## 一、更新 python

```
目前CentOS7.2自带的python版本是python2.7.5。由于yum这个软件需要系统自带的python工作。
如果冒然用自己安装的python替换掉系统自带的，可能造成yum不工作。
网上教程提示先安装 readline-devel ，openssl-devel安装包，我安装后发现电脑自带了，如未安装可以运行以下命令安装
```

```
yum install -y readline-devel
yum install -y openssl-devel
```

下载 python 并解压

```
cd /tmp
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
tar -xJvf Python-2.7.12.tar.xz
```

编译

```
cd Python-2.7.12/
./configure --prefix=/usr/local/python2.7
make
make install
```

链接

```
ln -s /usr/local/python2.7/bin/python2.7 /usr/local/bin/python
```

由于系统自带的 python 路径是/usr/bin/python。PATH 中，/usr/local/bin 比/usr/bin 靠前，所以当你输入 python，系统会自动启动你安装的 python2.7.12。

```
echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

## 二、安装 easy_install,pip

CentOS 安装 easy_install 的方法：

```
wget -q http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
```

CentOS 安装 python 包管理安装工具 pip 的方法如下：
注意：wget 获取 https 的时候要加上：--no-check-certificate

```
wget --no-check-certificate https://github.com/pypa/pip/archive/1.5.5.tar.gz
tar zvxf 1.5.5.tar.gz    #解压文件
cd pip-1.5.5/
python setup.py install
```

## 三、安装 twisted

```
pip install twisted
```

报错：no module named bz2
解决办法：

```
yum install bzip2*
```

然后重新编译 python,重新安装 easy_install,pip。
测试：

```
Python 2.7.12 (default, Jun  4 2017, 11:19:07)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-11)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import twisted
>>>
```

成功。
