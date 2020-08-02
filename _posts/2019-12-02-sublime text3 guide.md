---
layout: post
title: Sublime Text3 自救指南
subtitle: false
tags: [Tools]
---

<!-- ## sublime text3 自救指南 -->

打开Sublime Text3 ，点击菜单栏的`Preferences`–>`Package Control`,或者可以使用快捷键`CTRL+SHIFT+P` 打开

在打开的终端窗口，输入`install`,下方就会提示`Package Control:install package`,用鼠标点击

这时候等待几秒，就会弹出一个终端，在终端输入你想要安装的插件

在这个地方可以看到已安装的插件

## 汉化：

进入preference--packge control，点击install packge，搜索chinese localization，安装搜到的第一个插件即可。

## 个性化设置：

通过`Package Control`安装`PackageResourceViewer`插件

安装成功后 快捷键 `ctrl+shift+p` 输入 `PackageResourceViewer` 找到后选择`Open Resource`打开 ，选择 `Theme-default`

打开 `Theme-default` 修改 默认的主题配置

```txt
 {
     "class": "sidebar_container",
     // 侧边栏背景颜色
     "layer0.tint": [120, 120, 120],
     // "layer0.tint": [235, 237, 239],
     "layer0.opacity": 1.0,
     "content_margin": 0,
},
```

