---
title: 2018-06-25-django + gunicorn + nginx部署
urlname: dc4097dbbb7f8aef537fe72c595c2b38
date: '2022-12-09 20:02:28 +0800'
tags: []
categories: []
---

<hr />
<p>author: zyqzyq

date: 2018-06-25 9:16:32+00:00

layout: post

title: django + gunicorn + nginx 部署

categories: django

key: 20180625

tags:</p>

<ul>
<li>django</li>
<li>gunicorn</li>
<li>nginx</li>
</ul>
<hr />
<h1>创建python 虚拟环境</h1>
<h3>安装pipenv</h3>
<pre><code>sudo pip install pipenv 
</code></pre>
<h3>新建项目</h3>
<pre><code>mkdir ~/myproject
cd ~/myproject
</code></pre>
<h3>新建python环境</h3>
<pre><code>pipenv --python 3.6

</code></pre>

<blockquote>
<p>此处指定python版本进行安装，需要提前安装python3.6 ，也可以直接运行 pipenv --three 进行安装</p>
</blockquote>
<h3>激活环境并安装所需的包</h3>
<pre><code>pipenv shell
pipenv install django gunicorn celery
</code></pre>
<blockquote>
<p>至此环境配置完成。</p>
</blockquote>
<h1>新建并配置django项目</h1>
<h3>新建django 项目</h3>
<p>进入python 虚拟环境（以后默认都在该环境下进行操作）</p>
<pre><code>cd ~/myproject
pipenv shell
</code></pre>
<p>新建项目</p>
<pre><code>django-admin.py startproject myproject
</code></pre>
<h3>更改项目配置</h3>
<p>编辑myproject/setting 文件</p>
<pre><code>nano myproject/settings.py
</code></pre>
<p>在文件末尾添加以下内容并保存</p>
<pre><code>STATIC_ROOT = os.path.join(BASE_DIR, "static/")
</code></pre>
<h3>初始化项目</h3>
<p>初始化数据</p>
<pre><code>python manage.py makemigrations
python manage.py migrate

</code></pre>

<p>新建管理员账号</p>
<pre><code>python manage.py createsuperuser
</code></pre>
<p>收集静态文件</p>
<pre><code>python manage.py collectstatic
</code></pre>
<p>静态文件会被集中放置到~/myproject/static 中。</p>
<p>最后使用django自带的轻量级服务进行测试</p>
<pre><code>python manage.py runserver 0.0.0.0:8000
</code></pre>
<p>在浏览器中输入以下地址</p>
<pre><code>http://server_domain_or_IP:8000
</code></pre>
<p>你可以看到默认的django主页面</p>
<h3>测试gunicorn</h3>
<p>输入以下命令测试gunicorn</p>
<pre><code>gunicorn --bind 0.0.0.0:8000 myproject.wsgi:application
</code></pre>
<p>在浏览器中再次浏览网页，如果现实正常则说明gunicorn运行成功。</p>
<h1>新建gunicorn服务</h1>
<p>在文本编辑器中使用sudo权限创建并打开Gunicorn的启动文件</p>
<pre><code>sudo nano /etc/init/gunicorn.conf
</code></pre>
<p>首先为我们的服务添加一个简单的描述，然后定义自动启动服务的系统运行级别</p>
<pre><code>description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

</code></pre>

<p>接下来，我们要设置当服务停止时自动重启，并在指定的用户和组下运行，并将执行目录更改为我们的项目目录</p>
<pre><code>description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
setuid user
setgid www-data
chdir /home/user/myproject
</code></pre>

<p>现在，添加启动gunicorn的命令，我们需要给出虚拟环境中gunicorn可执行文件的路径。我们选择使用了unix套接字和nginx进行通信，相比较于网络端口这更安全，更快速。</p>
<pre><code>description "Gunicorn application server handling myproject"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
setuid user
setgid www-data
chdir /home/user/myproject

exec myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/user/myproject/myproject.sock myproject.wsgi:application
</code></pre>

<p>保存文件，然后启动gunicorn服务</p>
<pre><code>sudo service gunicorn start
</code></pre>
<h1>配置nginx代理gunicorn</h1>
<p>新建nginx配置文件</p>
<pre><code>sudo nano /etc/nginx/sites-available/myproject
</code></pre>
<p>在文件中，新建一个新的服务器，监听80端口并对服务器的ip或者域名做出响应。</p>
<pre><code>server {
    listen 80;
    server_name server_domain_or_IP;
}
</code></pre>
<p>接下来，我们需要设置忽略favicon相关的错误，然后设置静态文件的根目录。</p>
<pre><code>server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }

}
</code></pre>

<p>最后，我们会创建一个location / {}匹配所有其他请求，将流量通过套接字传递到gunicorn进程。</p>
<pre><code>server {
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
</code></pre>

<p>保存文件。通过文件链接来启动该文件</p>
<pre><code>sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
</code></pre>
<p>测试nginx 配置</p>
<pre><code>sudo nginx -t
</code></pre>
<p>如果没有错误，重启nginx服务</p>
<pre><code>sudo service nginx restart
</code></pre>
<p>大功告成。</p>
<p>参考网址：<a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn" target="_blank">https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04#configure-nginx-to-proxy-pass-to-gunicorn</a></p>
