---
title: 如何利用 Chrome 开发者工具寻找性能瓶颈
date: 2019-02-13 17:14:44
tags: 
  - 性能优化
  - 翻译
---

 ![How to Use Chrome Dev Tools to Find Performance Bottlenecks](https://scotch-res.cloudinary.com/image/upload/w_900,q_auto:good,f_auto/v1546777463/uvuyrkt4axev8q02umtg.png)

原文作者：[Jordan Irabor](https://scotch.io/@JordanIrabor)

译者：Sangle


当一个人在软件开发职业生涯中不断进步时，除了简单地编写有效的代码之外，还需要处理其他问题。在 web 开发的世界中，不仅要构建功能软件，还要使它们具有高性能，以便能够在使用最少资源的情况下实现用户所需的体验，这一点非常重要。

通常来说，这是一项非常艰巨的任务。因为如果没有工具来模拟和测量各种参数，对应用程序的资源消耗属性进行测试就不那么容易。

在这篇文章中，我们将深入了解这种工具：Chrome 开发者工具。具体来说，我们将详细了解 Audit 和 Performance 选项卡在评估 web 应用程序性能以及发现其存在的性能问题方面的灵活性。

为了使这个例子更加真实，我们将通过各种技术测试，来研究 Scotch.io 的性能问题并解决它们。

<!-- more -->

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774132/m9kno3ssyk1ajbumrvr3.png)

本文是[这篇文章](https://scotch.io/preview/browser-rendering-optimizations-for-frontend-development)的后续内容，您可能需要阅读这篇文章，以便轻松理解本文中提出的一些解决方案。



## 要求

要继续学习本教程，您需要在计算机上安装 Google Chrome 浏览器，并以隐身模式打开 scotch.io 网站，以便能够进行参考和尝试。 我们将以隐身模式工作，以确保浏览器处于干净状态，并且我们的测试不受已安装扩展的“噪音”的影响。



## 什么是性能 ？

正如[维基百科](https://en.wikipedia.org/wiki/Web_performance)上简单描述的那样，网页性能是指[网页](https://en.wikipedia.org/wiki/Web_page)加载和显示在用户浏览器上的速度。

在提高网站/ Web应用程序的性能时，需要考虑两个主要方面：

- 加载性能
- 运行性能

在本文中我们将更加关注加载性能。 加载性能只是指页面加载时的性能。 我们的主要目标是确定影响应用程序速度整体用户体验的性能问题。



## 开始

遵循下面的步骤：

1. 打开一个隐身模式的标签页( Control+Shift+N )

2. 在地址栏输入 [**https://scotch.io**](https://scotch.io/) 并打开 Chrome 开发者工具，使用快捷键 Command+Option+I (Mac) 或 Control+Shift+I (Windows, Linux)

3. 将开发者工具部分固定在右侧，如下图所示，以方便使用。

4. 切换到Audits选项卡，这是我们第一个要研究的工具。 借助此工具，我们将创建一个基准，后续对应用程序进行的更改都将与基准对比进行衡量，并了解哪些相关的修改能改进我们的应用程序。

   **Audits 选项卡可能藏在更多面板的箭头按钮后面**

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774172/z6jyu9qr8n0ytzvwirtv.png)

请记住，我们的主要目标是寻找 Scotch 的性能瓶颈并对其进行优化以获得更好的性能。

> 在软件工程中，瓶颈是指应用程序的负载受到单个因素的限制。就像瓶颈会减缓瓶子的整体水流一样。



## 加载性能

在本节中，我们将研究一些优化加载性能的方法。

## Audit

在使用 audit 进行评估时，我们使用了一个名为 Lighthouse 的工具。它是一个开源的自动化工具，可以优化网页的性能，包括表现、易读性、先进网页功能 (progressive web feature) 等。

在开发者工具的 Audits 选项卡里，我们可以对审查工具进行配置，面板上有以下这些配置项。

### Device

这使我们可以在移动端和桌面端之间切换用户代理。截至 2018 年第三季度，超过一半的网络流量来自移动设备，因此我们将对 Scotch.io 进行移动端审查。

### Audits

此设置允许我们选择要评估和改进的应用程序的质量。在这种情况下，性能是我们唯一关心的，因此我们可以取消所有其他选项。

### Throttling

此设置项使我们能够模拟在移动设备上浏览的条件。在这里，我们勾选快速3G, 4x CPU减速选项。事实上这并不会在审查期间进行节流，但是，它将有助于计算在相应的移动端条件下加载页面所需的时间。

### Clear Storage

该选项允许我们清除测试页面所有缓存的数据和资源，以便模拟访客首次访问网站时的体验，这里我们需要勾上这个选项。


按照上面指定的方式配置 Audit 之后，单击 Run audit 并等待10 - 30秒。它将对目标站点出具详细的性能报告。



## 解析 Audit 分析报告

当我们完成 Audit 分析时，会获得一份像这样的报告：

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774246/gbxd6stum2btsnqr2w8r.png)

右上角圈内的数字是综合所有项的得分，满分为100，我们现在拿到了55分，说明还有很大的优化空间。究竟要如何优化呢，让我们逐一查看报告的各项结果。

在 **Metrics 部分**，我们可以看到衡量网页性能的几个维度。

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774267/h0qvxfn2kcfrkfnwqvhg.png)

紧接着是一组屏幕截图，显示了从页面从初始状态到完全加载的各种 UI 状态：

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774288/hzierfkojzrgxqj5p1jl.png)

在 **opportunites** 部分中，我们可以看到为提高网站的加载性能而采取的具体步骤:

![img](https://d2mxuefqeaa7sj.cloudfront.net/s_3E9FB15F4931EDDD0FF74A78B944B652D05441745F0E6A34EAA9B809FD4A3B21_1546476875845_image.png)

Diagnostics 部分为我们提供了额外的性能信息，通常是有关决定web页面加载时间的因素:

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774337/jkdoglnvjpyd422pevfk.png)

最后，**Passed audits** 部分显示了该页面通过的性能检查，万岁!

![img](https://scotch-res.cloudinary.com/image/upload/dpr_2,w_800,q_auto:good,f_auto/v1546774389/h60lzryknwwdkumhngor.png)

## 优化方案

在这一节中，我们将进一步研究报告，以确定网页上的性能瓶颈，并提出可能的解决方案。

### Metrics 部分

在这个部分，有 5 个性能问题被高亮了，让我们看看可能的修复方式。

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774410/hs1oiwmw9viwxfgaywhy.png)

#### 第一次渲染

第一次有意义的渲染告诉我们页面的主要内容何时变得可视可用。根据报告，我们大约需要 **3.4秒** 才能看到主要内容。这可以通过单击 **View Trace** 按钮来确认。在 **Performance** 选项卡中，我们可以查看加载期间的各种UI状态，以确认在每个特定时间发生了什么。

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774430/ucxje9hs2ko6ivki8qtk.png)



注意在此时页面内容变得可见，如上图所示。

> 提示:你可以通过将鼠标悬停在每个高亮的指标上来获得更多信息。

**优化方案**

为了改善这一点，我们必须优化页面/整个应用程序的关键呈现路径。这意味着我们根据用户的需要优先显示内容，以便创建更好的体验和提高性能。可以通过减少关键资源的数量、关键路径长度和关键字节的数量来实现。点击[这里](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/optimizing-critical-rendering-path)了解更多。

#### Speed Index

这显示了页面可见内容的填充速度。这大约需要 7.2 秒，如 performance 选项卡所示:

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774472/u0tvci3ogzycnq11fogs.png)

**优化方案**

解决这个问题的一种方法是优化关键的呈现路径，这和前面提到的指标一致。第二种方法是优化内容效率。这包括手动删除不必要的文件下载，通过压缩优化传输编码，以及在任何可能的情况下进行缓存，以防止重新下载不变的资源。

#### CPU第一次空闲

它也被称为 First Interactive，它告诉我们页面何时满足最小交互需求（例如CPU有足够的空闲来处理用户输入、点击、滑动等)。从分析结果来看，这大约需要 6.5 秒。将这个值减到最小总是非常必要的:

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774526/mggxc8ztoptlyauzffin.png)

**优化方案**


为了提高这项指标的数据，我们需要采取与 Speed Index 相同的步骤。

#### Time to Interactive

这是页面实现完全交互所需的时间。我们的结果显示，这项时间高达 6.9 秒。在这种情况下的交互性描述如下:

- 页面显示了有意义的内容。
- 事件处理程序是为页面上大多数可见元素注册的。
- 页面能够在50毫秒内响应用户交互。

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774638/ad9g0dugj7j7kcorim6t.png)

**优化方案**

要修复这个问题，我们需要延迟或删除在页面加载期间发生的不必要的 JavaScript 。这通常可以通过代码分割和延迟加载来实现。压缩删除未使用的代码和缓存只发送用户需要的代码也十分必要。你可以在[这里](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/javascript-startup-optimization/)查看更多。

### 预估输入延迟


这描述了应用程序对用户输入的响应能力。此指标上的结果记录显示是大约 170 毫秒。应用程序通常有 100 毫秒响应用户输入，但是 Lighthouse 的目标是50毫秒。造成这种差异的原因是 Lighthouse 使用代理度量，即主线程的可用性来评估该指标而不是直接测量它。


一旦花费的时间超过了指定的时间，应用程序可能会被认为是滞后的。你可以在[这里](https://developers.google.com/web/tools/lighthouse/audits/estimated-input-latency)了解更多。

**优化方案**

为了改进这项指标，我们可以使用 service worker 来执行一些计算，从而释放主线程。另一个有用的方法是重构 CSS 选择器，以确保它们执行更少的计算。

### Opportunities 部分

正如在 **opportunities** 部分中高亮的一样，我们可以通过以下手段提高性能：

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_500,q_auto:good,f_auto/v1546774663/oswdedp0nvxxtutxkpf9.png)

### Avoiding multiple, costly round trips to any origin 避免多次，昂贵的往返任何来源

通常，当web页面加载时，它们连接到其他源以接收或发送数据。在这种情况下要提高性能，最好告知浏览器在渲染过程中提前建立到其他域名的链接，从而减少等待DNS查找和重定向的时间。

**优化方案**

我们可以通过在链接标签中添加rel 属性来告知浏览器我们使用这些资源的意图，如下图所示:

```html
<link rel="preconnect" href="https://scotchresources.com">
```

> 注意:在安全连接上，这仍然需要一些时间，因此必须在10秒内使用，否则浏览器将自动关闭连接，所有预连接工作都将浪费。

## 总结

我们现在已经成功的获得了一份关于 Scotch.io 的性能报告。使用 **Audit** 工具确定性能瓶颈，并确定对应的优化方案。

**加载**不是一个单一的时间点——它是一种没有任何一个度量标准可以完全概括的概念。在加载过程中，有多个时刻会影响用户认为它是“快”还是“慢”。

> 试试你自己的网站，让我们知道你发现了什么!

性能表现就像一列长长的火车，有许多不同的车厢，但目的是一致的。在测试中，你必须注意那些能够逐渐提高应用程序速度并为最终用户带来更好体验的小成就。


对于进一步阅读，[谷歌开发人员站点的Web basics](https://developers.google.com/web/fundamentals/)部分是一个很好的资源。

 