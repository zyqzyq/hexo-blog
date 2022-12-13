---
title: 语雀+hexo+github+vercel实现博客
urlname: sf325xe8rottnalb
date: '2022-12-13 22:25:06 +0800'
layout: post
categories:
  - blog
  - hexo
tags:
  - blog
  - hexo
---

# 准备工作

1. 注册语雀，github, vercel
2. 搜一个喜欢的 hexo 主题 clone 到自己仓库
3. vercel 选择该仓库部署
4. 这已经初步完成 hexo 的自动部署，下面就来打通语雀

# 同步语雀文章

新增语雀知识库，到知识库设置页面获取路径，待会要用

安装 yueque-hexo
相关链接：[https://npmmirror.com/package/yuque-hexo/v/1.2.1](https://npmmirror.com/package/yuque-hexo/v/1.2.1)

```
npm i -g yuque-hexo
```

修改项目下 package.json
其中 login 和 repo 改为刚刚设置页面看到的[https://www.yuque.com/login/](https://www.yuque.com/api/)repo

```
"yuqueConfig": {
    "postPath": "source/_posts",
    "cachePath": "yuque.json",
    "mdNameFormat": "title",
    "adapter": "hexo",
    "concurrency": 5,
    "baseUrl": "https://www.yuque.com/api/v2",
    "login": "yinzhi",
    "repo": "blog",
    "onlyPublished": false,
    "onlyPublic": false,
    "lastGeneratePath": "lastGeneratePath.log",
    "imgCdn": {
      "enabled": false,
      "concurrency": 0,
      "imageBed": "qiniu",
      "host": "",
      "bucket": "",
      "region": "",
      "prefixKey": ""
    }
  }
```

出于对知识库安全性的调整，使用第三方 API 访问知识库，需要传入环境变量 YUQUE_TOKEN，在语雀上点击 个人头像 -> 设置 -> Token 即可获取。传入 YUQUE_TOKEN 到 yuque-hexo 的进程有两种方式：

- 设置全局的环境变量 YUQUE_TOKEN
- 命令执行时传入环境变量
  - mac / linux: YUQUE_TOKEN=xxx yuque-hexo sync
  - windows: set YUQUE_TOKEN=xxx && yuque-hexo sync

执行命令后查看 source/\_posts 文章是否同步即可

# github action 自动同步

打开博客仓库 -> actions -> set up a workflow yourself

```
name: sync posts

# Controls when the action will run.
on:
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:
  repository_dispatch:
    types: [sync_event]
jobs:
  blog-cicd:
    name: sync Hexo blog
    runs-on: ubuntu-latest
    env:
      TZ: Asia/Shanghai
    steps:
    - name: Checkout codes
      uses: actions/checkout@v2

    - name: Setup node
      uses: actions/setup-node@v1
      with:
        node-version: '12.x'
    - name: Cache node modules
      uses: actions/cache@v1
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

    - name: sync yueque
      run: |
        npm install yuque-hexo -g
        YUQUE_TOKEN=${{secrets.YUQUE_TOKEN}} yuque-hexo sync
    - name: Commit & push changes
      run: |
        git config --global user.name "GitHub Action"
        git config --global user.email "action@github.com"
      	git add .
        git commit -m "GitHub Actions Auto Sync Post at $(date +'%Y-%m-%d %H:%M:%S')"
        git push
```

需要提前配置 YUQUE_TOEKN
这边触发方式选择了 2 个，1 手动触发，2 外部触发，手动触发方便调试，外部触发可以用接口触发
相关文档：[https://docs.github.com/en/actions/using-workflows/triggering-a-workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)
调试完能自动拉文章即可

# 语雀自动触发同步

因为语雀的 webhook 不能传参，所以这里选择 vercel 搭了个 serveless 进行转发，有需求可以直接拉代码然后用 vercel 部一个
仓库地址：[https://github.com/zyqzyq/blog-webhook](https://github.com/zyqzyq/blog-webhook)

语雀添加 webhook

- 打开设置
- 消息推送
- 其他渠道
- 随便配个机器人名字然后添加 url 例如 http://xxx.vercel.com/dispatch_github?user=aaa&repo=hexo-blog&token=xxx&event_type=sync_event
- 然后添加
- 点击验证，结束

# 快速操作

1. hexo+github+vercel 原先就有
2. 注册语雀，添加知识库，导入文章
3. 获取语雀 toekn, login, repo
4. package.json 添加 yuqueconfig
5. github 仓库添加 actions
6. fork 云函数仓库然后语雀 webhook 调接口 或者 github action 选择定时触发
7. 大功告成
