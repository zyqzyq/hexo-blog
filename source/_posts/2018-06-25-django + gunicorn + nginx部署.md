---
title: django + gunicorn + nginx 部署
urlname: dc4097dbbb7f8aef537fe72c595c2b38
date: '2018-06-25 09:16:32 +0800'
layout: post
categories: django
tags:
  - django
  - gunicorn
  - nginx
---

# 创建 python 虚拟环境

### 安装 pipenv

```
sudo pip install pipenv
```

### 新建项目

```
mkdir ~/myproject
cd ~/myproject
```

### 新建 python 环境

```
pipenv --python 3.6
```

> 此处指定 python 版本进行安装，需要提前安装 python3.6 ，也可以直接运行 pipenv --three 进行安装

### 激活环境并安装所需的包

```
pipenv shell
pipenv install django gunicorn celery
```

> 至此环境配置完成。

# 新建并配置 django 项目

### 新建 django 项目

进入 python 虚拟环境（以后默认都在该环境下进行操作）

```
cd ~/myproject
pipenv shell
```

新建项目

```
django-admin.py startproject myproject
```

### 更改项目配置

编辑 myproject/setting 文件

```
nano myproject/settings.py
```

在文件末尾添加以下内容并保存

```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

### 初始化项目

初始化数据

```
python manage.py makemigrations
python manage.py migrate
```

新建管理员账号

```
python manage.py createsuperuser
```

收集静态文件

```
python manage.py collectstatic
```

静态文件会被集中放置到~/myproject/static 中。
最后使用 django 自带的轻量级服务进行测试

```
python manage.py runserver 0.0.0.0:8000
```

在浏览器中输入以下地址

```
http://server_domain_or_IP:8000
```

你可以看到默认的 django 主页面

### 测试 gunicorn

输入以下命令测试 gunicorn

```
gunicorn --bind 0.0.0.0:8000 myproject.wsgi:application
```

在浏览器中再次浏览网页，如果现实正常则说明 gunicorn 运行成功。

# 新建 gunicorn 服务

在文本编辑器中使用 sudo 权限创建并打开 Gunicorn 的启动文件

```
sudo nano /etc/init/gunicorn.conf
```

首先为我们的服务添加一个简单的描述，然后定义自动启动服务的系统运行级别

```
description "Gunicorn application server handling myproject"
start on runlevel [2345]
stop on runlevel [!2345]
```

接下来，我们要设置当服务停止时自动重启，并在指定的用户和组下运行，并将执行目录更改为我们的项目目录

```
description "Gunicorn application server handling myproject"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
setuid user
setgid www-data
chdir /home/user/myproject
```

现在，添加启动 gunicorn 的命令，我们需要给出虚拟环境中 gunicorn 可执行文件的路径。我们选择使用了 unix 套接字和 nginx 进行通信，相比较于网络端口这更安全，更快速。

```
description "Gunicorn application server handling myproject"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
setuid user
setgid www-data
chdir /home/user/myproject
exec myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/user/myproject/myproject.sock myproject.wsgi:application
```

保存文件，然后启动 gunicorn 服务

```
sudo service gunicorn start
```

# 配置 nginx 代理 gunicorn

新建 nginx 配置文件

```
sudo nano /etc/nginx/sites-available/myproject
```

在文件中，新建一个新的服务器，监听 80 端口并对服务器的 ip 或者域名做出响应。

```
server {
    listen 80;
    server_name server_domain_or_IP;
}
```

接下来，我们需要设置忽略 favicon 相关的错误，然后设置静态文件的根目录。

```
server {
    listen 80;
    server_name server_domain_or_IP;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }
}
```

最后，我们会创建一个 location / {}匹配所有其他请求，将流量通过套接字传递到 gunicorn 进程。

```
server {
    listen 80;
    server_name server_domain_or_IP;
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/user/myproject/myproject.sock;
    }
}
```

保存文件。通过文件链接来启动该文件

```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```

测试 nginx 配置

```
sudo nginx -t
```

如果没有错误，重启 nginx 服务

```
sudo service nginx restart
```

大功告成。
参考网址：[https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn)
