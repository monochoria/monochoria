---
layout: post
title: Linux 调换 command 与左 ctrl 键
category: diary
---



1. 在 home 目录下建立 .xmodmaprc 文件
2. 向文件内写入如下内容

```
! 解除现有的绑定

clear Control

clear Mod4

! 将 key 37 (左 ctrl) 映射到 Super_L (i.e. 'cmd')

keycode  37 = Super_L

! 将 key 133 (左 cmd) 映射到 Control_L (i.e. 'ctrl)

keycode 133 = Control_L

! 更新设置

add control = Control_L

add mod4    = Super_L
```

3. 运行 `xmodmap ~/.xmodmaprc`

