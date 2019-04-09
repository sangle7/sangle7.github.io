---
title: What's New In DevTools (Chrome 74)
date: 2019-04-09 11:20:44
tags: 
  - chrome
  - devtool
---


> 原文链接： https://developers.google.com/web/updates/2019/03/devtools

Chrome 74 的开发者工具有了以下更新：

## 高亮显示受CSS属性影响的所有节点

将鼠标悬停在影响节点盒模型的 css 属性上方，例如 padding 或 margin，可以看到所有被这个属性影响的节点

![Hovering over a margin property highlights all nodes affected by that             declaration](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1.png)

**图1**.将鼠标悬停在margin属性上会突出显示受该属性影响的所有节点的 margin


<!-- more -->


## Audit 面板中的 Lighthouse v4

新的[点击目标审查](https://developers.google.com/web/tools/lighthouse/audits/tap-targets)现在会检查出那些没有适当放大并且间隔开，从而导致不合适在移动设备上点击的按钮和链接等交互元素。

![The tap targets audit](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/2.png)

**Figure 2**.点击目标审查

报告中的的 PWA 部分现在使用了徽章评分系统

![The new badge scoring system for the PWA category](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/3.png)

**图 3**. PWA 新的徽章评分系统

> **Lighthouse 是什么?** [Lighthouse](https://developers.google.com/web/tools/lighthouse) 是为 Audits 面板提供支持的审计引擎。它还支持[web.dev/measure](https://web.dev/measure) 和 [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/). 您还可以将 Lighthouse 作为 Node模块, Chrome 扩展程序或命令行运行。参阅 [Chrome DevTools 中的 RunRun Lighthouse ](https://developers.google.com/web/tools/lighthouse/#devtools)开始使用。

## WebSocket 二进制消息查看器

查看 WebSocket 的二进制消息：

1. Open the **Network** panel. See [Inspect Network Activity](https://developers.google.com/web/tools/chrome-devtools/network/) to learn the basics of analyzing network activity.打开 **Network** 面板，在 [Inspect Network Activity](https://developers.google.com/web/tools/chrome-devtools/network/) 中我们可以看到网络活动的基本分析

   

   ![The Network panel](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/4.png)**图4**. Network 面板

   

2. 点击 **WS** 来过滤掉不是 WebSocket 的连接

   

   ![After clicking WS only WebSockety connections are shown](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/5.png)

   **图 5**. 点击 WS 后只显示 WebSocket 连接

   

3. 在 **Name** 栏点击一个 WebSocket 连接的来详细查看

   

   ![Inspecting a WebSocket connection](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/6.png)

   **图 6**. 查看一个webscorkt连接

   

4. 点击 **Messages** 标签.

   

   ![The Messages tab](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/7.png)

   **图 7**. Messages 标签

   

5. 点击其中一个 **Binary Message** 来查看它

   

   ![Inspecting a binary message](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/binary4.png)

   **Figure 8**.查看二进制消息

   

利用查看器底部的下拉菜单，可以将消息转化为Base64或UTF-8格式。点击 **Copy to clipboard** 拷贝二进制消息。

![Viewing a binary message as Base64](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/8.png)

**图 9**. 通过 Base64 格式查看二进制消息

## 在命令菜单中获取区域屏幕截图 

通过区域屏幕截图，你可以捕获视口的一部分的屏幕截图。 这个功能已经存在了一段时间，但访问它的入口非常隐蔽。 而现在，区域屏幕截图可以从命令菜单中获得。

1. 打开开发者工具，然后按Control + Shift + P或Command + Shift + P（Mac）打开命令菜单。

   ![The Command Menu](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/9.png)

   **图 10**. 命令菜单

   

2. 输入 `area`, 选择 **区域屏幕截图**, 回车

3. 将鼠标拖到要截屏的视图部分上进行截取

   ![Selecting the portion of viewport to screenshot](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/10.png)

   **图 11**.选择要截屏的视图部分

   

## Network 面板中的Service worker 过滤器

在 Network 面板过滤器文本框中输入以下内容：service-worker-initiated 或 is：service-worker-intercepted，以查看 service worker 导致（已启动）或可能已修改（截获）的请求。

![Filtering by is:service-worker-initiated](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/11.png)

**图 12**. 通过 `is:service-worker-initiated`过滤

![Filtering by is:service-worker-intercepted](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/12.png)

**图 13**. 通过 `is:service-worker-intercepted`过滤

查看更多关于network面板过滤器的信息：[过滤资源](https://developers.google.com/web/tools/chrome-devtools/network/#filter)



## Performance 面板更新

性能录制现在标记出了长任务和首次渲染。

有关使用 Performance 面板分析页面加载性能的示例，请查看[减少主线程进行的工作](https://developers.google.com/web/tools/chrome-devtools/speed/get-started#main)。



## 性能录制中的长任务

性能录制现在会标记出长任务

![Hovering over a long task in a Performance recording](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/13.png)

**图 14**. 光标悬停在一个性能录制中的长任务上



## Timings 部分的首次渲染

Performance 中的 [Timings 部分](https://developers.google.com/web/updates/2018/11/devtools#metrics) 现在标记出了首次渲染的时间

![First Paint in the Timings section](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/14.png)

**图 15**. Timings 部分的首次渲染



## 新的DOM教程

查看[开始查看和更改DOM](https://developers.google.com/web/tools/chrome-devtools/dom/)，以便亲身体验与DOM相关的新功能。