---
title: 为 Scratch 添加绿米扩展
tags: [WoT,Scratch,CodeLab,Aqara,智能家居,绿米]
date: 2020-08-11 10:00:00
---
>种瓜上周末与 Aqara 开放平台工程师、CEO、几位董事以及其他的小伙伴碰了个面，一起讨论在 本月 18-19 号的 Aqara Home 中国服务商大会上 使用 CodeLab 创作平台做哪些有趣的演示，以及后续的合作可能: Aqara(绿米)智能家居用户(尤其是孩子)可在 CodeLab 创作平台上对智能设备进行编程，让人们将智能家庭改造为魔法世界吧！

最近一周，我使用 ThingTalk 完成了 CodeLab 与 Aqara 的对接，这个 Scratch 扩展将会包含在 CodeLab 的下一次发布中，同时会出现在 Aqara 8 月中旬的服务商大会上。

CodeLab Neverland 里使用了不少的 Aqara 设备，并基于这些设备构建了许多好玩的实验，这些案例以及 Scratch Aqara 扩展的使用说明，我就偷懒直接从种瓜的博客转载过来了：

欢迎来到霍格沃茨：

<video style="width:50%" controls>
  <source src="https://adapter.codelab.club/video/wand.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

在 Scratch 中构建控制房间里一切事物的 Siri:

<video style="width:100%" controls>
  <source src="https://adapter.codelab.club/video/1593328552202841.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

下班打个响指关闭所有电灯

<video style="width:100%" controls>
  <source src="https://adapter.codelab.club/video/turnofflight_byfinger.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

使用魔杖打开窗帘：

<video style="width:100%" controls>
  <source src="https://adapter.codelab.club/video/1576905337206766.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

训练一个 AI 管家，它了解你的生活习惯，当你打开书时，自动为你把书房的灯打开；当你合上书思考时，为你关闭电灯，让你沉浸在黑暗中思考:

<video style="width:100%" controls>
  <source src="https://adapter.codelab.club/video/%E8%AF%BB%E4%B9%A6%E4%B8%8E%E6%80%9D%E8%80%83.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>

更多有趣案例请自行访问 CodeLab 的 [项目展示页](https://adapter.codelab.club/user_guide/gallery/)

## 使用方式
首先打开 CodeLab 创造平台的扩展页
![](https://adapter.codelab.club/img/6f6fc565cbcff4105784b448c628c307.png)

### 获取 token
点击`打开绿米授权`积木，将打开登陆页面，使用 Aqara 账号登陆后，将获得一个 token。

### 连接到云
之后将 token 复制到连接积木里, 运行它即可。
![](https://adapter.codelab.club/img/d9aefea95691b05581de0c8382168af4.png)

### hello world
构建一个入门程序: 当小猫被点击时，将灯泡打开
![](https://adapter.codelab.club/img/ca0019da61b7c540b340665bed5c0c5e.png)

## 参考
[发布 CodeLab Adapter3.5](https://blog.just4fun.site/post/%E5%B0%91%E5%84%BF%E7%BC%96%E7%A8%8B/release-3-5/#aqara)