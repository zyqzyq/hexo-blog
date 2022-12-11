---
title: 2017-10-09-仿开眼APP-kotlin
urlname: f8d4469307d677134407c61ba4e55a9a
date: '2022-12-09 20:02:19 +0800'
tags: []
categories: []
---

<hr />
<p>layout: post

title: 仿开眼 APP kotlin

date: 2017-10-09 16:25:06

categories: kotlin

key: 20171019

tags:</p>

<ul>
<li>kotlin</li>
</ul>
<hr />
<h1>Eyepetizer</h1>
<pre><code>主要是为了通过仿写APP更好的学习kotlin(选择该APP原因主要是因为有大佬已经写过了，站在巨人的肩膀站得高看的远)
</code></pre>
<p><a href="https://github.com/zyqzyq/Eyepetizer-kotlin" target="_blank">gitHub地址</a></p>
<h1>计划(基本完成)</h1>
<pre><code>主页
发现页面（包含热门，分类，作者）
关注页面
播放页面   
全部作者页面
全部分类页面
分类详情页面（包含首页，全部，作者，专辑）
排行榜页面（包含周排行，月排行，总排行）
搜索页面
</code></pre>
<h1>目前进度</h1>
<h2>启动页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/splash.png" alt class="align-none" /></p>
<pre><code>开启启动画面渐变 ( Handler+Thread )
</code></pre>
<h2>首页</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/home.png" alt class="align-none" /></p>
<pre><code>显示每日精选自动轮播自动播放5秒小视频介绍 (viewpager + indicator)(增加无限循环，优化最后一页跳转卡顿)
显示推荐视频选项（简单的添加显示在recyclerView中 ）（每日精选的视频右下角添加图片标识）
实现每日精选文字逐字显示
实现下拉放大图片刷新
增加再按一次退出提示
</code></pre>
<h2>播放页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/play1.png" alt class="align-none" />

<img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/play2.png" alt class="align-none" /></p>

<pre><code>旋转和点击控制全屏播放
实现显示作品相关信息(暂未实现缓存功能)
实现相关视频推荐
</code></pre>
<h2>发现页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/discoverHot.png" alt class="align-none" />

<img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/discoverCategory.png" alt class="align-none" /></p>

<pre><code>实现热门小页面
实现banner轮播图（用的git大佬的轮子,链接在最底下）
实现热门视频推荐
实现热门排行链接（横向的recyclerView实现）
实现分类小页面
页面的item主要用的banner轮子(有一些细微的改动)
实现作者小页面
使用横向的recyclerView实现最新作者推荐栏的滑动
</code></pre>
<h2>关注页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/follow.png" alt class="align-none" /></p>
<pre><code>主要调用之前的fragment 快速实现
</code></pre>
<h2>我的页面</h2>
<pre><code>主要就显示显示（准备实现缓存功能）
</code></pre>
<h2>全部作者页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/pgcsAll.png" alt class="align-none" /></p>
<pre><code>与发现作者小页面一样
</code></pre>
<h2>全部分类页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/categoryAll.png" alt class="align-none" /></p>
<pre><code>使用gridView显示分类列表（不知道热门排行，热门专题，360全景的api就没添加）
</code></pre>
<h2>分类详情页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/categoryDetail.png" alt class="align-none" /></p>
<pre><code>scrollView + tabLayout + viewPager + Fragment  实现4个小分页的显示
</code></pre>
<h2>排行榜页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/rankList1.png" alt class="align-none" />

<img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/rankList2.png" alt class="align-none" /></p>

<pre><code>使用和发现页面类似的方法，由于子页面数据类型一样，用同一个fragment实现。
</code></pre>
<h2>搜索页面</h2>
<p><img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/search1.png" alt class="align-none" />

<img src="https://raw.githubusercontent.com/zyqzyq/Eyepetizer-kotlin/master/screenshots/search2.png" alt class="align-none" /></p>

<pre><code>偷懒使用了activity + recyclerView简单实现
</code></pre>
<h2>bug</h2>
<pre><code>状态栏无法完全透明
</code></pre>
<h2>TODO</h2>
<pre><code>准备实现缓存功能
准备优化界面显示
</code></pre>
<h1>实现方式</h1>
<pre><code>mvp 框架
okhttp+retrofit+rxjava 实现网络请求框架
TabLayout+Fragment 实现底部导航栏
TabLayout + ViewPager + Fragment 实现分页显示
</code></pre>
<h1>关于我</h1>
<pre><code>联系：497533265@qq.com    
</code></pre>
<h2>声明</h2>
<pre><code>Api 数据都是来自开眼视频，数据接口均属于非正常渠道获取，请勿用于商业用途，原作公司拥有所有权利。
</code></pre>
<h2>参考</h2>
<pre><code>https://github.com/kaikaixue/Eyepetizer
https://github.com/LRH1993/Eyepetizer-in-Kotlin
https://github.com/youth5201314/banner
https://github.com/CarGuo/GSYVideoPlayer
感谢大佬们的资源，向大佬们学习。
</code></pre>
