---
title: 2017-06-05-kivy-textinput弹出输入框处理
urlname: 19a7b695efb6540f13e693701abb1d4f
date: '2022-12-09 20:02:14 +0800'
tags: []
categories: []
---

<hr />
<p>layout: post

title: kivy textinput 弹出输入框处理

date: 2017-06-05 16:25:06

categories: kivy

key: 20170605-1

tags:</p>

<ul>
<li>kivy</li>
</ul>
<hr />
<p>在kivy打包Android程序后运行，点击输入框会自动弹出输入法将自己的控件遮挡住，在群友（校长巨佬）的指导下找到了softinput_mod 参数，可以控制输入法的弹窗模式。</p>
<blockquote>
<p>softinput_mode 官网参数介绍：

This specifies the behavior of window contents on display of the soft keyboard on mobile platforms. It can be one of ”, ‘pan’, ‘scale’, ‘resize’ or ‘below_target’. Their effects are listed below.</p>

</blockquote>
<p>”. The main window is left as is, allowing you to use the keyboard_height to manage the window contents manually.

‘pan’. The main window pans, moving the bottom part of the window to be always on top of the keyboard.

‘resize’. The window is resized and the contents scaled to fit the remaining space.

‘below_target’. The window pans so that the current target TextInput widget requesting the keyboard is presented just above the soft keyboard.</p>

<p>'scale'参数在官网未找到相关说明

'below_target'需要 kivy1.9.1 版本或以上</p>

<p>例如：</p>
<pre><code>`from kivy.core.window import WindowBase

WindowBase.softinput_mode='below_target'`
</code></pre>

<p>softinput_mode默认为‘’</p>
