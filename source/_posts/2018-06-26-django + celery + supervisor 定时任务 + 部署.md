---
title: 2018-06-26-django + celery + supervisor 定时任务 + 部署
urlname: c338b2d897ca073247963da6d032b6b1
date: '2022-12-09 20:02:29 +0800'
tags: []
categories: []
---

<hr />
<p>author: zyqzyq

date: 2018-06-26 16:16:32+00:00

layout: post

title: django + celery + supervisor 定时任务 + 部署

categories: django

key: 20180626

tags:</p>

<ul>
<li>django</li>
<li>celery</li>
<li>supervisor</li>
</ul>
<hr />
<h2>环境</h2>
<pre><code>django 2.0.6 (上一篇文章有详细项目新建的描述)
celery 4.2.0
django-celery-beat 1.1.1
django-celery-results 1.0.1
</code></pre>
<h1>安装配置celery</h1>
<h3>选择broker</h3>
<pre><code>我选择了RabbitMQ(官方推荐，安装配置简单) [了解更多](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html#choosing-a-broker)
安装RabbitMQ
</code></pre>
<pre><code>sudo apt-get install rabbitmq-server
</code></pre>
<p>添加配置到setting.py末尾</p>
<pre><code># celery broker
BROKER_URL = 'amqp://localhost//'

</code></pre>

<h3>安装celery</h3>
<pre><code>celery是一个python包所以可以通过pip直接安装
</code></pre>
<pre><code>cd ~/user/myproject
pipenv shell
pipenv install celery 
</code></pre>
<pre><code>celery 3.1版本以后不需要再安装django-celery模块。
</code></pre>
<p>添加配置到setting.py末尾</p>
<pre><code>
CELERY_ACCEPT_CONTENT = ['pickle']
CELERY_TASK_SERIALIZER = 'pickle'
CELERY_RESULT_SERIALIZER = 'pickle'

# celery 时区设置，使用 settings 中 TIME_ZONE 同样的时区

CELERY_TIMEZONE = TIME_ZONE

</code></pre>

<h3>安装django-celery-results</h3>
<pre><code>django-celery-results使用django orm提供结果。
</code></pre>
<p>安装 django-celery-restults</p>
<pre><code>pip install django-celery-results

</code></pre>

<pre><code>添加 django-celery-restults 到setting.py
</code></pre>
<pre><code>INSTALLED_APPS = (
    ...,
    'django_celery_results',
)
</code></pre>
<pre><code>添加配置到setting.py末尾
</code></pre>
<pre><code># celery结果返回
CELERY_RESULT_BACKEND = 'django-db'

</code></pre>
<h3>安装django-celery-beat</h3>
<p>自定义调度程序类可以在命令行中指定（--scheduler参数）。</p>
<p>默认调度程序是celery.beat.PersistentScheduler，它只是跟踪本地搁置数据库文件中的最后一次运行时间。</p>
<p>还有一个django-celery-beat扩展，它将计划存储在Django数据库中，并提供了一个方便的管理界面来管理运行时的定期任务。</p>
<p>安装 django-celery-beat</p>
<pre><code>pip install django-celery-beat

</code></pre>

<p>添加 django-celery-beat 到setting.py</p>
<pre><code>INSTALLED_APPS = (
    ...,
    'django_celery_beat',
)
</code></pre>
<p>添加配置到setting.py末尾</p>
<pre><code># celery 定时任务
CELERY_BEAT_SCHEDULER = "django_celery_beat.schedulers:DatabaseScheduler"

</code></pre>

<p>启动任务</p>
<pre><code>celery -A proj beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
</code></pre>
<p>访问Django后台界面设置定期任务。</p>
<h1>添加任务</h1>
<p>新建一个项目</p>
<pre><code>python manage.py startapp demoapp
</code></pre>
<p>添加demoapp/tasks.py

@shared_task 修饰器可以让你创建任务而不需要任何具体的应用程序实例</p>

<pre><code># Create your tasks here
from __future__ import absolute_import, unicode_literals
from celery import shared_task


@shared_task
def add(x, y):
    return x + y


@shared_task
def mul(x, y):
    return x * y


@shared_task
def xsum(numbers):
    return sum(numbers)
</code></pre>
<h1>supervisor 配置</h1>
<p>pip 安装supervisor (建议用系统自带的python2进行安装，py3不确定是否支持)</p>
<pre><code>pip install supervisor
</code></pre>
<p>生成配置文件</p>
<pre><code>echo_supervisord_conf > /etc/supervisord.conf
</code></pre>
<p>编辑配置文件，在末尾添加</p>
<pre><code>[program:celery.worker]
command= myprojectenv/bin/celery -A myproject worker -l info
directory=/home/user/myproject
numprocs=1
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/home/home/user/myproject/log/celery_worker_out.log

[program:celery.beat]
command= myprojectenv/bin/celery -A myproject beat -l info
directory=/home/user/myproject
numprocs=1
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/home/home/user/myproject/log/celery_beat_out.log

</code></pre>

<p>启动supervisor</p>
<pre><code>supervisord -c supervisord.conf
</code></pre>
<p>至此，大功告成。

django 的配置可以按上一篇文章进行配置。</p>

<p>参考文件：

<a href="http:_docs.celeryproject.org_en_latest_django_first-steps-with-django" target="_blank">http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html</a></p>

<p><a href="http:_docs.celeryproject.org_en_latest_getting-started_first-steps-with-celery" target="_blank">http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html</a></p>
