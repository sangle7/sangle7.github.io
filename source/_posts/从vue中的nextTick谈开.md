---
title: 从vue中的nextTick谈开
date: 2019-03-06 16:50:44
tags: 
  - javascript
---

最近面试中提到手写一个符合标准的简单 Promise，鉴于是面试， then 中只要用settimeout 就可以，这里就引发我的思考，如何在浏览器环境模拟 microtask 呢？

这里就想到vue中的nextTick，可以模拟 microtask 并提供了 macrotask 的降级方案，查看 vue 源码后发现在 vue 中的 nextTick 的实现采用优雅降级的 4 种方式：setImmediate、MessageChannel、Promise 和 setTimeout。(此前vue曾使用过 MutationObserver ，由于MO 在 IOS9.3 以上的 WebView 中有bug，当有了更好的替代方案时 vue 果断放弃了mo)

<!-- more -->

作者在源码中给出了如下的注释：

> Here we have async deferring wrappers using both microtasks and (macro) tasks. In < 2.4 we used microtasks everywhere, but there are some scenarios where microtasks have too high a priority and fire in between supposedly sequential events (e.g. #4521, #6690) or even between bubbling of the same event (#6566). However, using (macro) tasks everywhere also has subtle problems when state is changed right before repaint (e.g. #6813, out-in transitions). Here we use microtask by default, but expose a way to force (macro) task when needed (e.g. in event handlers attached by v-on).

大概意思就是 nextTick 方法使用了 microtask 和 (macro) task。低于 2.4 的版本中到处都用 microtask，但有些场景下因为 microtask 优先级太高执行的太早了，然而用 (macro) task 也有一些小问题。所以，默认还是使用 microtask，但提供一种方式在必要的情况下可以使用 (macro) task。

## microtask 与 (macro) task
注释中提到了两个名词：microtask 和 (macro) task，这两个是属于 JavaScript Event Loop（事件循环）中的概念。上面提到的 Promise（浏览器原生实现）、MutationObserver、MessageChannel 属于 microtask，setImmediate 和 setTimeout 属于 (macro) task。

分类：
- (macro) task：script（整体代码）、setTimeout、setInterval、 setImmediate、requestAnimationFrame、I/O、UI Rendering。
- microtask：process.nextTick、Promise、Object.observe、MutationObserver。


**推荐阅读**
https://github.com/Ma63d/vue-analysis/issues/6
https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/

