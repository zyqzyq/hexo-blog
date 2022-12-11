---
title: kivy textinput弹出输入框处理
urlname: 19a7b695efb6540f13e693701abb1d4f
date: '2017-06-05 16:25:06 +0800'
layout: post
categories: kivy
tags:
  - kivy
---

在 kivy 打包 Android 程序后运行，点击输入框会自动弹出输入法将自己的控件遮挡住，在群友（校长巨佬）的指导下找到了 softinput_mod 参数，可以控制输入法的弹窗模式。

> softinput_mode 官网参数介绍：
> This specifies the behavior of window contents on display of the soft keyboard on mobile platforms. It can be one of ”, ‘pan’, ‘scale’, ‘resize’ or ‘below_target’. Their effects are listed below.

”. The main window is left as is, allowing you to use the keyboard_height to manage the window contents manually.
‘pan’. The main window pans, moving the bottom part of the window to be always on top of the keyboard.
‘resize’. The window is resized and the contents scaled to fit the remaining space.
‘below_target’. The window pans so that the current target TextInput widget requesting the keyboard is presented just above the soft keyboard.
'scale'参数在官网未找到相关说明
'below_target'需要 kivy1.9.1 版本或以上
例如：

```
`from kivy.core.window import WindowBase
WindowBase.softinput_mode='below_target'`
```

softinput_mode 默认为‘’
