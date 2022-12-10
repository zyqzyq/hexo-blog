---
title: 2018-07-18-django Translation 实现中英文切换
urlname: ea56a08a3843799212555d85eb6c097c
date: '2022-12-09 20:02:30 +0800'
tags: []
categories: []
---

<hr />
<p>author: zyqzyq

date: 2018-07-18 10:16:32+00:00

layout: post

title: django Translation 实现中英文切换

categories: django

key: 20180718

tags:</p>

<ul>
<li>django</li>
</ul>
<hr />
<h2>环境</h2>
<pre><code>django 2.0.6 (之前文章有详细项目新建的描述)
</code></pre>
<h2>配置</h2>
<h3>setting.py</h3>
<h6>配置 MIDDLEWARE</h6>
<pre><code>MIDDLEWARE = [
   'django.contrib.sessions.middleware.SessionMiddleware',
   'django.middleware.locale.LocaleMiddleware',
   'django.middleware.common.CommonMiddleware',
]
</code></pre>
<p>注意：LocaleMiddleware 需要放在SessionMiddleware之后，CommonMiddleware之前。

<a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#how-django-discovers-language-preference" target="_blank">详细介绍</a></p>

<h6>配置template</h6>
<pre><code>TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.i18n',
            ],
        },
    },
]
</code></pre>
<h6>启用i18n</h6>
<pre><code>USE_I18N = True

USE_L10N = True

USE_TZ = True

# 翻译文件所在目录，需手工创建

LOCALE_PATHS = [
os.path.join(BASE_DIR, 'locale'),
]

LANGUAGE_CODE = 'zh-hans'

LANGUAGES = {
('zh-hans', '中文简体'),
('en', 'English'),
}
</code></pre>

<p>注意：如果local文件夹在app目录下无需创建LOCALE_PATHS变量。<a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#how-django-discovers-translations" target="_blank">详细介绍</a></p>
<p>LANGUAGES 中可以添加多个django支持的语言（这边以中英文为例）</p>
<h3>yourproject/urls.py</h3>
<pre><code>from django.conf.urls.i18n import i18n_patterns
urlpatterns = [
    path('i18n/', include('django.conf.urls.i18n')),
]

urlpatterns += i18n_patterns(
path('', include('yourapp.urls')),
)
</code></pre>

<h2>声明需要翻译的字符串</h2>
<h5>准备工作</h5>
<p>需要提前安装gettext

在 ubuntu 下运行</p>

<pre><code>sudo apt-get install gettext
</code></pre>
<p>即可。</p>
<h3>在python代码中</h3>
<p>在此示例中，文本“欢迎访问我的网站”。 被标记为翻译字符串：</p>
<pre><code>from django.http import HttpResponse
from django.utils.translation import gettext as _

def my*view(request):
output = *("欢迎访问我的网站")
return HttpResponse(output)
</code></pre>

<p><a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#internationalization-in-python-code" target="_blank">更多介绍</a></p>
<h3>在tmplate代码中</h3>
<p>需要将{％ load i18n ％}

放到模板文件头部（继承的文件也需要假如该声明）</p>

<p>{％trans％}模板标记转换为常量字符串（用单引号或双引号括起来）或变量内容：</p>
<pre data-syntax="python"><code class="lang-python hljs raw"><title>{％ trans "This is the title." ％}</title>
<title>{％ trans myvar ％}</title>
</code></pre>
<p>如果您的翻译需要带变量的字符串（占位符），请改用{％blocktrans％}。</p>
<pre><code>{％ blocktrans ％}This string will have {{ value }} inside.{％ endblocktrans ％}
</code></pre>
<p>如果您想要检索已翻译的字符串而不显示它，可以使用以下语法：</p>
<pre><code>{％ trans "This is the title" as the_title ％}

<title>{{ the_title }}</title>
<meta name="description" content="{{ the_title }}">
</code></pre>
<p><a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#internationalization-in-template-code" target="_blank">更多介绍</a></p>
<h2>创建语言文件</h2>
<p>要创建或更新消息文件，请运行以下命令：</p>
<pre><code>django-admin makemessages -l en

</code></pre>

<p>en 是你想创建的文件的语言。</p>
<p>注意：该脚本应该从两个地方之一运行（在运行前需要新建locale文件夹）：</p>
<pre><code>Django项目的根目录（包含manage.py的目录）。
您的一个Django应用程序的根目录。
</code></pre>
<p>运行完命令后会创建一个.po文件

编辑 yourproject/locale/en/LC_MESSAGES/django.po</p>

<pre><code>#: path/to/python/module.py:23
msgid "欢迎访问我的网站。 "
msgstr "Welcome to my site."
</code></pre>
<p>msgid 是标记的想要翻译的字符串

msgstr 是你想要翻译后显示的字符串，默认为空。</p>

<p><a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#localization-how-to-create-language-files" target="_blank">更多介绍</a></p>
<h3>编译语言文件</h3>
<p>运行以下代码</p>
<pre><code>django-admin compilemessages
</code></pre>
<p>大功告成。</p>
<p><a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#compiling-message-files" target="_blank">详细介绍</a></p>
<h2>设置语言</h2>
<p>在模板中添加以下代码</p>
<pre><code>{％ load i18n ％}

<form action="{％ url 'set_language' ％}" method="post">{％ csrf_token ％}
    <input name="next" type="hidden" value="{{ redirect_to }}" />
    <select name="language">
        {％ get_current_language as LANGUAGE_CODE ％}
        {％ get_available_languages as LANGUAGES ％}
        {％ get_language_info_list for LANGUAGES as languages ％}
        {％ for language in languages ％}
            <option value="{{ language.code }}"{％ if language.code == LANGUAGE_CODE ％} selected{％ endif ％}>
                {{ language.name_local }} ({{ language.code }})
            </option>
        {％ endfor ％}
    </select>
    <input type="submit" value="Go" />
</form>
</code></pre>
<p>实现语言选择功能。

<a href="https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/translation/#the-set-language-redirect-view" target="_blank">更多介绍</a></p>

<p>以上是简单的实现中英文切换的步骤，如需了解更多，可以点击更多介绍进入官网进行了解。</p>
<p>提示：复制后请把%替换为小写。</p>
