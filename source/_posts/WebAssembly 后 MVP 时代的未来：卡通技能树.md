---
title: WebAssembly 后 MVP 时代的未来：卡通技能树
date: 2018-10-30 10:14:44
tags: 
  - WebAssembly
  - 翻译
  - javascript
---

开发者可能普遍对 WebAssembly 还接触的不多，文末有关于 WebAssembly 的展开阅读

> 原文链接：[WebAssembly’s post-MVP future: A cartoon skill tree](https://hacks.mozilla.org/2018/10/webassemblys-post-mvp-future/)

人们对 WebAssembly 有误解，他们认为于 2017 年登陆浏览器的 WebAssembly MVP 版本就是 WebAssembly 的最终版本。

我可以理解这种误解来自哪里， WebAssembly 社区组实际上致力于向后兼容。 这意味着你今天创建的 WebAssembly **未来将能够**继续在浏览器上工作。

但这并不意味着目前的 WebAssembly 功能完整。 事实上，这远非如此。 WebAssembly 还有许多功能，它们将从根本上改变你使用 WebAssembly 所能做的事情。

我认为这些未来的功能有点像电子游戏中的技能树。 我们已经完全掌握了这些技能中的前几项，但是我们还需要完成下面的整个技能树来解锁完整的应用程序。

<!-- more -->

![显示WebAssembly技能的技能树，将在帖子的过程中填写。](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-09-final-e1539904436477-500x319.png)

那么，让我们来看看已经掌握的内容，然后我们就可以看到未来需要的内容了。

## 最小可行性版本（Minimum Viable Product）

![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-00-A-1-e1539904807805.png)



WebAssembly 故事开始于 [Emscripten](http://kripken.github.io/emscripten-site/) ，它可以通过将 C ++ 代码转换为JavaScript 来在 web上运行 C ++ 代码。 这使得将大型现有的 C ++ 代码库（如游戏和桌面应用程序）引入 web 成为可能。

但它自动生成的 JS 仍然比原生代码慢得多。 但 Mozilla 工程师发现了一个隐藏在生成的 JavaScript 中的类型系统，并想出了如何[使这个JavaScript运行得非常快](https://blog.mozilla.org/luke/2014/01/14/asm-js-aot-compilation-and-startup-performance/)。 这个 JavaScript 子集名为[asm.js。](https://hacks.mozilla.org/2018/10/webassemblys-post-mvp-future/asmjs.org)

其他浏览器供应商看到 asm.js 的速度非常快，所以他们也开始向他们的引擎[添加相同的优化](https://hacks.mozilla.org/2015/03/asm-speedups-everywhere)。

但那并不是故事的结局。 这只是一个开始。 引擎还有一些可以做的优化，这可以让它更快。

但他们无法在 JavaScript 本身中做到这一点。 相反，他们需要一种新的语言，专门为编译而设计的语言。 这就是 WebAssembly。

那么第一版 WebAssembly 需要哪些功能？ 我们需要什么才能获得可以在 web 上有效运行 C 和 C ++ ？

### 编译目标

![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-01-comp-target-e1539905023913-500x255.png)

WebAssembly 的开发者不只希望支持 C 和 C ++。 他们希望能够将许多不同的语言编译为 WebAssembly。 所以他们需要一个与语言无关的编译目标。

他们需要类似汇编语言的东西，像桌面应用程序这样的东西被编译成 x86。 但是这种汇编语言不适用于实际的物理机器。 这将是一个概念机器。

### 快速运行

![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-02-fast-exec-e1539905310659-500x254.png)

必须设计该编译目标，以便它可以非常快地运行。 否则，在Web上运行的WebAssembly 应用程序无法满足用户对平滑交互和游戏的期望。

### 紧凑

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-03-compact-e1539905278906-500x254.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-03-compact-e1539905278906.png)

除了运行时间，加载时间也需要快。 用户对某些内容的加载速度有一定的期望。 对于桌面应用程序，由于应用程序已安装在你的计算机上，因此它们会快速加载。对于 Web 应用程序，用户期望加载时间也很快，因为 Web 应用程序通常不必加载与桌面应用程序一样多的代码。

但是，当你将这两件事结合起来时，它会变得棘手。 桌面应用程序通常是相当大的代码库。 因此，如果它们在网络上，当用户第一次访问URL时，需要下载和编译很多内容。

为了满足这些期望，我们需要我们的编译目标是紧凑（体积小）的。 这样，它可以快速通过网络。

### 线性内存

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-04-linear-memory-e1539905361396-500x255.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-01-SS-04-linear-memory-e1539905361396.png)

这些语言还需要能够以不同于 JavaScript 使用内存的方式使用内存。 他们需要能够直接管理他们的内存——指示哪些字节在一起。

这是因为像 C 和 C ++ 这样的语言有一个叫做指针的低级特性。 你可以拥有一个没有值，而是具有该值的内存地址的变量。 因此，如果要支持指针，程序需要能够从特定地址进行写入和读取。

但是你不能让你从网上下载的程序随意访问内存中的字节。因此，为了创建一种安全的方式来访问内存，就像使用本机程序一样，我们必须创建一种能够访问内存特定部分的东西。

为此，WebAssembly 使用线性内存模型。 这是使用 TypedArrays 实现的。 它基本上就像一个 JavaScript 数组，但这个数组只包含内存字节。 当你访问其中的数据时，你只需使用数组索引，你可以将它们视为内存地址。 这意味着你可以假装这个数组是 C ++ 内存。

### 成就解锁

因此，利用所有这些技能，人们可以在浏览器中运行桌面应用程序和游戏，就好像它们在计算机上运行一样。

这几乎就是 WebAssembly MVP 发布时的技能。 它确实是最小的可行性版本。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-03-final-e1539905426663-500x432.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-01-mvp-03-final-e1539905426663.png)

这允许某些类型的应用程序工作，但仍有许多其他应用程序可以解锁。

## 重量级桌面应用程序

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-00-A-e1539905812771-500x277.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-00-A.png)

下一个需要解锁的成就是更重的桌面应用程序。

你能想象像 Photoshop 这样的东西在你的浏览器中运行吗？ 你可以像使用Gmail 一样在任何设备上即时地加载它？

这样的事情已经逐渐成为现实。 例如，Autodesk 的 AutoCAD 团队已将其 CAD 软件用于浏览器，Adobe 已经使用 WebAssembly 通过浏览器提供了 Lightroom。

但是我们还需要新增一些功能，以确保所有这些应用程序——即使是最复杂的应用程序——在浏览器中运行良好。

### 线程

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-01-threading-e1540219281254-500x162.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-01-threading-e1540219281254.png)

首先，我们需要支持多线程。 现代计算机有多个核心。 这基本上是多个大脑，多个线程可以同时工作。 这可以使事情变得更快，为了利用这些核心，WebAssembly 需要支持线程。

### SIMD

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-02-SIMD-e1540219323375-500x160.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-02-SIMD-e1540219323375.png)

除了线程之外，还有另一种利用现代硬件的技术，它可以让你并行处理事物。

那就是SIMD：单指令多数据。 使用SIMD，可以占用大块内存，并分成不同的执行单元，这类似于内核。 然后你有相同的代码 —— 在这些执行单元上运行相同的指令，但它们每个都将该指令应用于它们自己的数据位。

### 64位寻址

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-03-wasm64-e1540219360998-500x160.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-01-SS-03-wasm64-e1540219360998.png)

WebAssembly 需要充分利用的另一个硬件功能是 64 位寻址。

内存地址只是数字，所以如果你的内存地址只有 32 位长，你只能拥有 4GB 的线性内存。

但是对于 64 位寻址，你有 16EB。 当然，你的计算机中没有 16EB 的实际内存。 因此，最大值取决于系统实际可以提供的内存大小，而不是取决于 WebAssembly。

### 流式编译

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-01-streaming-e1539905897570-500x365.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-01-streaming-e1539905897570.png)

对于这些应用程序，我们不仅需要它们快速运行。同时，我们需要的加载时间要比现在更快。 我们需要一些专门作用于改善加载时间的优化。

一个重要的步骤是进行流式编译——在仍在下载的同时编译 WebAssembly 文件。WebAssembly 专门设计了用于实现容易的流式编译。 在 Firefox 中，我们编译它的速度比通过网络下载的[速度快得多](https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/)，在下载文件时，它几乎完成了编译。 其他浏览器也在逐渐支持该功能。



![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-01-streaming-e1539905897570-500x365.png)

另一件重要的事情是拥有一个分层编译器。

在 Firefox 中，这意味着有[两个编译器](https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/ )。 第一个，称为基础编译器(base compiler)，在文件开始下载后立即启动。 它可以非常快速地编译代码，以便快速启动。

它生成的代码很快，但并没有优化到 100%。 为了获得更好的性能，我们在后台的几个线程上运行另一个编译器——优化编译器(optimizing compiler)。 这个编译需要更长的时间，但生成极快的代码。 完成后，我们会将基础版本换成完全优化的版本。

这样，我们可以使用基础编译器快速启动，并使用优化编译器快速执行。

此外，我们正在开发一种名为 [Cranelift](https://github.com/CraneStation/cranelift) 的新型优化编译器。 Cranelift 旨在函数级别上并行地快速编译代码。同时，它生成的代码比我们当前的优化编译器具有更好的性能。

Cranelift 目前已经添加到 Firefox 的开发版本，但默认情况下已禁用。 一旦我们启用它，我们将更快地获得完全优化的代码，并且代码将运行得更快。

但是我们可以使用更好的技巧来实现它，所以我们大多数时间都不需要编译......

### 隐式HTTP缓存

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-03-http-e1540219501952-500x359.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-03-http-e1540219501952.png)

使用WebAssembly，如果在两个页面加载时加载相同的代码，它将编译为相同的机器代码。 它不需要根据流经它的数据进行更改，就像 JS JIT 编译器需要做的那样。

这意味着我们可以将编译后的代码存储在 HTTP 缓存中。 然后，当页面加载并去获取 .wasm 文件时，它将只从缓存中提取预编译的机器代码，而不用进行任何编译。

### 其他改进

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-04-other-e1540219569386-500x359.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-02-SS-04-other-e1540219569386.png)

目前有许多围绕其他方面改进的讨论正在进行，请继续关注其他有关加载时间的改进。

### 我们进展如何？

为了支持这些重量级应用程序，我们现在的进展如何呢？

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-03-P-07-e1540219610872-500x359.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-03-P-07-e1540219610872.png)

- 线程

  关于线程，我们有[一个提案](https://github.com/WebAssembly/threads)已经完成了，但是今年早些时候[在浏览器中关闭了](https://blog.mozilla.org/security/2018/01/03/mitigations-landing-new-class-timing-attack/)一个关键功能 - [SharedArrayBuffers](https://hacks.mozilla.org/2017/06/a-cartoon-intro-to-arraybuffers-and-sharedarraybuffers/) 。  它们将再次打开。 关闭它们只是一种临时措施，可以减少在今年早些时候发布的 CPU 中发现的幽灵安全问题的影响，这方面的工作还在进展中，敬请期待。

- SIMD

  [SIMD](https://github.com/WebAssembly/simd/blob/master/proposals/simd/SIMD.md) 目前正处于活跃的开发阶段。

- 64 位寻址

  对于 [wasm-64](https://github.com/WebAssembly/design/blob/master/FutureFeatures.md#linear-memory-bigger-than-4-gib) ，我们已经了解了如何添加它，它与 x86 或 ARM 支持 64 位寻址非常相似。

- 流式编译

  我们在 2017 年末添加了[流式编译](https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/) ，其他浏览器也在逐步跟进。

- 分层编译

  我们在2017年末添加了[基础编译器](https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/) ，其他浏览器在过去一年中添加了相同类型的架构。

- 隐式HTTP缓存

  在Firefox中，我们[接近](https://bugzilla.mozilla.org/show_bug.cgi?id=1487113)实现对隐式HTTP缓存的支持。

- 其他改进

  其他的改进目前正在讨论中。

这一切仍在进行中，但你已经可以看到，现在出现了一些重量级应用程序，因为WebAssembly 已经为这些应用程序提供了所需的性能。

一旦这些功能全部到位，将会有更多的重量级应用程序将能够进入浏览器。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-04-final-e1540219657102-500x453.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-02-heavyweight-04-final-e1540219657102.png)

## 与 JavaScript 互操作的小模块

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-00-A-500x281.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-00-A.png)

WebAssembly 不仅适用于游戏和重量级应用程序，也适用于常规 Web 开发：小型模块类型的 Web 开发。

有时你的应用程序的小角落会进行大量繁重的处理，在某些情况下，使用WebAssembly 可以更快地完成此处理。 我们想使这些小模块的部分更方便地移植到 WebAssembly。

这是一个已经发生的例子。开发人员已经将 WebAssembly 的模块合并到有很多小模块做大量繁重工作的地方。

一个例子是在 Firefox 的 DevTools 和 webpack 中使用的 source map 中的解析器。它被[用 Rust 重新编写](https://hacks.mozilla.org/2018/01/oxidizing-source-maps-with-rust-and-webassembly/)，编译为 WebAssembly，这使得它的速度提高了11倍。在进行相同的重写后，WordPress 的 Gutenberg 解析器[平均速度提高了86倍](https://mnt.io/2018/08/22/from-rust-to-beyond-the-webassembly-galaxy/) 。

但是，要真正广泛地使用这种用法——让人们在使用时感到舒服——我们还需要有更多的东西。

### JS 和 WebAssembly之间的快速调用

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-01-call-opts-e1540220093322-500x260.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-01-call-opts-e1540220093322.png)

首先，我们需要 JS 和 WebAssembly 之间的快速调用，因为如果你将一个WebAssembly 模块集成到现有的 JS 系统中，很有可能你需要在两者之间进行大量调用，而这些调用的速度需要很快。

但是，当 WebAssembly 第一次出现时，这些调用并不快。在 MVP 版本中，引擎对两者之间的调用只有最基本的支持。 引擎只是让调用可以生效，但并没有提高它的速度。因此，引擎需要在这方面进行优化。

我们最近在 Firefox 中完成了这方面的工作。目前，其中的一些调用事实上要[比非内联JavaScript调用更快。](https://hacks.mozilla.org/2018/10/calls-between-javascript-and-webassembly-are-finally-fast/) 而其他引擎也在努力研究这个问题。

### 快速简便的数据交换

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-02-data-exchange-e1540220133991-500x260.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-02-data-exchange-e1540220133991.png)

然而，这带给我们另一个问题。 当你在 JavaScript 和 WebAssembly 之间进行调用时，通常需要在它们之间传递数据。

你需要将值传递给 WebAssembly 函数或从中返回值。 这可能很慢，也可能很困难。

说这件事很难有几个原因。 其中的一个原因是，目前 WebAssembly 只能理解数字。这意味着不能将更复杂的值(比如对象)作为参数传入。你需要把那个对象转换成数字，并把它放入线性内存中。 然后将线性内存中的位置传递给 WebAssembly。

这有点复杂。 将数据转换为线性内存需要一些时间。 因此，我们需要这样将这个操作变得更容易，更快捷。

### ES模块集成

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-03-es-module-e1540220169128-500x259.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-03-es-module-e1540220169128.png)

我们还需要集成浏览器内置的 ES 模块支持。现在，使用命令式 API 实例化WebAssembly模块。你调用一个函数，它会返回一个模块。

但这意味着 WebAssembly 模块并不是 JS 模块图的一部分。为了像使用 JS 模块那样使用导入和导出，我们需要对 ES 模块进行集成。

### 工具链集成

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-04-toolchain-e1540220201137-500x260.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-04-toolchain-e1540220201137.png)

但是，只是能够导入和导出并不足够。 我们需要一个地方来分发这些模块，并从中下载它们，还需要一些工具来打包它们。

WebAssembly 的 npm 是什么？ 直接使用npm怎么样？

WebAssembly 的 webpack 或 parcel 是什么？直接使用 webpack 或者 parcel 怎么样？

对于使用者来说，WebAssembly 的这些模块和其他 JS 模块应该没有任何不同，所以没有理由创建一个单独的生态系统，我们只需要工具来集成它们。

### 兼容性

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-05-bc-compat-e1540220231412-500x259.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-01-SS-05-bc-compat-e1540220231412.png)

在现有的 JS 应用程序中，我们还需要做一件事——支持旧版本的浏览器，甚至是那些不知道 WebAssembly 是什么的浏览器。我们需要确保你不必为了支持 IE11 而用JavaScript 完全重新编写整个模块。

### 我们进展如何？

那目前我们的进展如何呢？

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-02-P-05-e1540220264112-500x257.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-02-P-05-e1540220264112.png)

- JS 和 WebAssembly 之间的快速调用

  [JS 和 WebAssembly 之间的调用现在在 Firefox 中很快](https://hacks.mozilla.org/?p=32717) ，其他浏览器也正在研究这个问题。

- 轻松快速的数据交换

  为了便于快速地进行数据交换，有一些建议可以帮助解决这个问题。

  正如我之前提到的，你必须使用线性内存来处理更复杂的数据类型的一个原因是因为 WebAssembly 只能理解数字。 它拥有的唯一类型是整数和浮点数。

  在 [引用类型提议](https://github.com/WebAssembly/reference-types/blob/master/proposals/reference-types/Overview.md) 中，这一点将会改变。该提议添加了一个新类型，WebAssembly 函数可以将其作为参数并返回。 此类型是对 WebAssembly 外部对象的引用——例如，JavaScript对象。

  但是 WebAssembly 无法直接操作这个对象。 要实际执行诸如调用方法之类的操作，它仍然需要使用一些 JavaScript 粘合剂。 这意味着它可以工作，但它比我们需要的慢。

  为了加快速度，有一项提议叫做[主机绑定提案](https://github.com/WebAssembly/host-bindings/blob/master/proposals/host-bindings/Overview.md) 。 它让一个 wasm 模块声明哪些粘合剂对于其导入和导出是必需的，这样这些粘合剂就不需要用 JS 编写了。 通过将 JS 中的粘合剂提取到 wasm 中，在调用内置 Web API 时，这些粘合剂可以完全被优化掉。

  还有一个我们可以简化的互动部分。这与跟踪数据需要在内存中停留多长时间有关。如果在线性内存中有一些数据是 JS 需要访问的，那么就必须将其留在那里，直到 JS 读取数据。但如果你把它永远留在那里，就会出现所谓的内存泄漏。如何知道何时可以删除数据？你怎么知道 JS 什么时候用完了？目前，你必须自己管理。

  一旦 JS 用完了数据，JS 代码就必须调用类似释放函数的东西来释放内存。 但这很容易出错。 为了简化此过程，我们将 [WeakRefs ](https://github.com/tc39/proposal-weakrefs)添加到 JavaScript。 有了这个，你能够观察 JS 上的对象。 然后，当该对象被垃圾收集时，你可以在WebAssembly 端进行清理。

  所以这些建议都在酝酿中。 与此同时， [Rust生态系统已经创建了一些工具](https://hacks.mozilla.org/2018/03/making-webassembly-better-for-rust-for-all-languages/) ，可以为你自动完成这一切，并且可以为正在提案中的部分提供降级方案。

  特别值得一提的是一种工具，[wasm-bindgen](https://rustwasm.github.io/wasm-bindgen/) ，其他语言也可以使用它。 当它看到你的 Rust 代码应该做类似接收或返回某些类型的 JS 值或 DOM 对象之类的东西时，它会自动创建为你做这件事的 JavaScript 粘合代码，所以你不需要考虑它。 而且因为它是以与语言无关的方式编写的，其他语言工具链也可以采用它。


- ES模块集成

  对于ES模块集成， [该提案](https://github.com/WebAssembly/esm-integration/tree/master/proposals/esm-integration)非常接近。 我们正在与浏览器供应商合作实施它。

- 工具链支持

  对于工具链支持，Rust 生态系统中有像 [`wasm-pack`](https://github.com/rustwasm/wasm-pack) 这样的工具，可以自动运行为 npm 打包代码所需的一切。 开发者也在积极致力于支持。

- 兼容性

  最后，为了向后兼容，我们还有`wasm2js`工具。 它获取一个 wasm 文件并输出等效的 JS 。这份 JS 不会很快，但至少这意味着它可以在不支持WebAssembly 的老版本浏览器上运行。

所以我们正在接近解锁这一成就。 一旦我们解锁它，我们将会开辟另外两条路径。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-03-final-e1540220313636-500x270.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-04-js-interop-03-final-e1540220313636.png)

## JS 框架和  compile-to-js 语言

一个是重写 WebAssembly 中的 JavaScript 框架等大部分内容。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-01-A-01-frameworks-e1540220354792-500x273.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-01-A-01-frameworks-e1540220354792.png)

另一个是使静态类型的 compile-to-js 语言可以编译为 WebAssembly —— 例如，将 [Scala.js](https://www.scala-js.org/) ， [Reason](https://reasonml.github.io/) 或 [Elm ](https://elm-lang.org/)等语言编译为 WebAssembly。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-01-A-02-langs-e1540220481271-500x275.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-01-A-02-langs-e1540220481271.png)

对于这两种用例，WebAssembly 都需要支持高级语言功能。

### GC

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-01-gc-e1540220514444-500x159.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-01-gc-e1540220514444.png)

出于几个原因，我们需要与浏览器的垃圾收集器集成。

首先，让我们看看重写 JS 框架的部分内容。 这可能有几个好处。 例如，在React中，你可以做的一件事是在 Rust 中重写 DOM diffing 算法，它具有非常符合人机工程学的多线程支持，并将该算法并行化。

你也可以通过不同的方式分配内存来加快速度。 在虚拟 DOM 中，你可以使用特殊的内存分配方案，而不是创建需要进行垃圾回收的一堆对象。 例如，你可以使用一个 bump 分配器方案，它具有非常低成本的一次性分配机制。这可能有助于加快速度和减少内存使用。 

但是你仍然需要与该代码中的 JS 对象（例如组件）进行交互。 你不能只是不断地将所有内容复制到线性内存中，因为这样做既困难又低效。

因此，你需要能够与浏览器的 GC 集成，以便你可以使用由 JavaScript VM 管理的组件。 其中一些 JS 对象需要指向线性内存中的数据，而有时候线性内存中的数据需要指向 JS 对象。

如果这最终创建了循环，那么垃圾收集器可能会出现问题。 这意味着垃圾收集器将无法判断对象是否已被使用，因此永远不会收集它们。 WebAssembly 需要与 GC 集成，以确保这些类型的跨语言数据的依赖性有效。

这也将帮助静态类型的语言编译成 JS，比如 Scala.js, Reason, Kotlin 或 Elm。这些语言在编译为 JS 时使用 JavaScript 的垃圾收集器。因为 WebAssembly 可以使用相同的GC （引擎内置的GC） ，这些语言将能够编译为 WebAssembly 并且能使用相同的垃圾收集器。 他们不需要改变 GC 的工作方式。

### 异常处理

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-02-exception-e1540220563466-500x155.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-02-exception-e1540220563466.png)

我们还需要对异常处理进行更好的支持。

有些语言，比如 Rust，是没有异常的。 但在其他语言中，例如C ++，JS或C＃，异常处理通常会被广泛使用。

目前可以通过 polyfill 来处理异常，但是 polyfill 使代码运行得非常慢。因此，在编译 WebAssembly 时，当前的默认情况是编译时没有异常处理。

但是，由于 JavaScript 有异常，即使你编译了代码而不运行它们，JS也可能会抛出一个异常。 如果 WebAssembly 函数调用抛出的 JS 函数，则 WebAssembly 模块将无法正确处理异常。 像 Rust 这样的语言会选择在这种情况下中止运行，我们需要对这个问题进行优化。

### 调试

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-03-debugging-e1540220592314-500x159.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-03-debugging.png)

使用 JS 和类 JS 语言的人们习惯的另一件事是良好的调试支持。 所有主流浏览器中的 Devtools 都可以轻松地对 JS 进行逐步调试。 我们需要同样级别的支持来在浏览器中调试 WebAssembly。

### 尾调用

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-04-tail-calls-e1540220637629-500x160.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-02-SS-04-tail-calls-e1540220637629.png)

最后，对于许多函数式语言，你需要支持[尾调用](https://en.wikipedia.org/wiki/Tail_call)。 我不打算对此进行详细讨论，但总体来说，它允许你调用一个新函数，而无需向堆栈添加一个新的调用记录。因此，对于支持此功能的函数式语言，我们希望 WebAssembly 也支持它。

### 我们进展如何？

那目前我们的进展如何呢？

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-03-P-04-e1540220662110-500x156.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-03-P-04.png)

- 垃圾收集

  对于垃圾收集，目前正在进行两项提案：

  JS的 [Typed Objects 提议](https://github.com/tschneidereit/typed-objects-explainer)和 WebAssembly 的[ GC 提议](https://github.com/WebAssembly/gc) 。 Typed Objects 可以描述对象的固定结构。 该提案将在即将召开的TC39会议上讨论

  WebAssembly GC提议可以直接访问该结构，该提案正在积极探讨中。

  有了这两个，JS 和 WebAssembly 都能知道对象的结构，并且可以共享该对象并有效地访问存储在其上的数据。 我们的团队已经有了这个方案的原型。 但是，将这些方案标准化仍需要一定的时间，我们可能会在明年的某个时候取得进展。

- 异常处理

  [异常处理 ](https://github.com/WebAssembly/exception-handling/blob/master/proposals/Level-1.md)仍处于研究和开发阶段，现在我们在看它是否可以有效利用其他提案，例如我之前提到的 Typed Objects 提案。

- 调试

  对于调试，目前在浏览器 devtools 中有一些支持。 例如，你可以在 Firefox 调试器中单步执行 WebAssembly 的文本格式。但它仍然不理想。 我们希望在调试时能够展示在源代码中的位置，而不是在编译后的代码中。我们需要做的就是弄清楚 source map ——或者类似  source map 的东西 ——在 WebAssembly 中应该如何工作。有一个 WebAssembly CG 的子组在研究这个。

- 尾调用

  [尾调用提案](https://github.com/WebAssembly/tail-call/blob/master/proposals/tail-call/Overview.md)也在进行中。

一旦完成这些，我们将解锁 JS 框架和许多 compile-to-JS 语言。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-04-final-e1540220704600-500x405.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-05-high-level-04-final.png)

这些都是我们可以在浏览器中解锁的成就。 但是在浏览器之外呢？

## 在浏览器之外

现在，当我谈到“浏览器之外”时，你可能会感到困惑。 因为浏览器不是用来查看网页的吗？ 就像我们的名称一样—— WebAssembly

但事实是你在浏览器中看到的东西 - HTML，CSS 和 JavaScript - 只是网络的一部分。它们是可见的部分——它们是用来创建用户界面的——因此它们是最明显的。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-02-browser-toolbox-e1540221152912-500x220.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-02-browser-toolbox-e1540221152912.png)

但是，网络中另一个非常重要的部分是不可见的。

这就是链接，这是一种非常特殊的链接。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-03-link-e1540220782538-500x216.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-03-link-e1540220782538.png)

这个链接的创新之处在于，我可以链接到你的页面，而不需要把它放在中央注册中心，也不需要问你，甚至不需要知道你是谁。我就可以把那个链接放在这里。

正是这种连接的简易性，没有任何监督或审批瓶颈，使我们的网络成为可能。 这正是我们与我们不认识的人建立这些全球社区的原因。

但如果我们所拥有的只是链接，那么我们有两个问题没有解决。

第一个问题是，你去访问这个网站，它会给你提供一些代码。它如何知道它应该为你提供什么样的代码？因为如果你在Mac上运行，那么你需要不同于Windows的代码。这就是为什么不同的操作系统有不同版本的程序。

那么一个网站是否应该为每个可能的设备提供不同版本的代码？不。

相反，该站点应该只有一个版本的代码——源代码。 这是提供给用户的内容。 然后它被转换为用户设备上的机器代码。

这个概念的名称是可移植性。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-portability-02-e1540220821857-500x340.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-portability-02-e1540220821857.png)

这很好，你可以从不认识你的人那里加载代码，而对方也不知道你正在运行什么样的设备。

但这带来了第二个问题。 如果你不认识这些你正在加载的网页的提供者，你怎么知道他们给你什么样的代码？它可能是恶意代码。它可能试图接管你的系统。

从任意其他人那里获得网络运行代码，难道不意味着你必须盲目地信任任何一个在网络上的人吗?

这是网络上另一个关键概念的用武之地。

这是安全模型。 我打算把它称为沙盒。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-security-02-e1540220859323-500x246.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-security-02-e1540220859323.png)

基本上，浏览器获取页面——其他人的代码——并不是让它在你的系统中肆无忌惮地运行，而是将它放在沙盒中。它会将一些不危险的玩具放入沙盒中，以便代码可以执行某些操作，但它会将危险的东西留在沙盒之外。

所以链接的效用基于以下两点：

- 可移植性——向用户提供代码并使其在可运行浏览器的任何类型的设备上运行的能力。
- 沙盒——一种安全模型，可让你运行该代码而不会危及机器的完整性。

那么为什么这种区别很重要呢？ 如果我们将 Web 视为浏览器使用 HTML，CSS 和 JS 向我们展示的东西，和如果我们从可移植性和沙盒方面考虑 Web，会有什么不同呢？

因为它改变了你对 WebAssembly 的看法。

你可以将 WebAssembly 视为浏览器工具箱中的另一个工具......事实的确如此。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-wasm-in-tb-1-e1540221053348-500x225.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-04-wasm-in-tb-1-e1540221053348.png)

它是浏览器工具箱中的另一个工具。但不仅仅是这样。它还为我们提供了一种方法，将网络的另外两个属性——可移植性和安全模型——带到其他需要它们的地方。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-06-expand-02-e1540220978397-500x219.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-intro-06-expand-02-e1540220978397.png)

我们可以将网络扩展到浏览器的边界。 现在让我们来看看这些网络属性的用途。

## Node.js

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-A-e1540221197730-500x277.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-A-e1540221197730.png)WebAssembly 如何帮助 Node ？ 它可以为 Node 带来完全的可移植性。

Node 为你提供了 浏览器中大部分 JavaScript 的可移植性。 但是在很多情况下，Node 的 JS 模块还不够，我们需要提高性能或重用现有的代码，而这些代码并非用 JS 编写。

在这些情况下，你需要 Node 的本地模块。 这些模块使用 C 之类的语言编写，需要针对用户运行的特定类型的机器进行编译。

本地模块可以在用户安装时进行编译，也可以在不同系统的广泛矩阵中预编译为二进制文件。 前一种方法对于用户来说是一种痛苦，后一种方法是对包维护者的痛苦。

现在，如果这些本地模块是用 WebAssembly 编写的，那么它们就不需要专门针对目标系统进行编译。 相反，他们只是像 Node 中的 JavaScript 一样运行。 但是他们几乎是以原生的方式来做的。

因此，我们为 Node 中运行的代码提供了完全的可移植性。 你可以使用完全相同的 Node 应用程序并在所有不同类型的设备上运行它，而无需编译任何内容。

但是 WebAssembly 无法直接访问系统的资源。 Node 中的本地模块不是沙盒，它们可以完全访问浏览器从沙盒中取出的所有危险玩具。 在 Node 中，JS 模块也可以访问这些危险的玩具，因为 Node 使它们可用。 例如，Node 提供了从系统读取和写入文件的方法。

在 Node 的例子中，模块具有对危险系统 api 的这种访问是有一定意义的。因此，如果 WebAssembly 模块默认没有这种访问功能，我们如何为 WebAssembly 模块提供所需的访问权限？ 我们需要传入函数，以便 WebAssembly 模块可以与操作系统一起工作，就像 Node 和 JS 一样。

对于 Node，可能包含很多像 C 标准库这样的功能。 它也可能包括 [POSIX](https://en.wikipedia.org/wiki/POSIX) 的一部分 ——便携式操作系统接口——这是一种较早的有助于兼容性的标准。 它提供了一个API，用于跨越一堆不同的类 Unix 操作系统与系统进行交互。 模块肯定需要一堆类似 POSIX 的功能。

### 便携式接口

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-SS-e1540221292681-500x194.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-SS-e1540221292681.png)

Node 核心人员需要做的是找出要暴露的函数集和要使用的API。

但如果这些是通用的，不是很好吗? 不是仅仅针对Node，也可以在其他运行时和用例中使用？

如果你愿意，可以使用 POSIX for WebAssembly。 或许是一个PWSIX —— 便携式WebAssembly系统接口。

如果以正确的方式完成，甚至可以为Web实现相同的API。 这些标准 API 可以填充到现有 Web API 上。

这些函数不属于 WebAssembly 规范的一部分。而且会有 WebAssembly 的主机无法使用它们。但是对于那些可以使用这些函数的平台，不管代码运行在哪个平台上，都将有一个统一的 API 来调用这些函数。这将使通用模块——同时在web和 Node 上运行的模块——变得更加容易。

### 我们进展如何？

那么，这是否真的会发生？

有几些事情正对这个想法有利。 有一个名为[包名映射](https://github.com/domenic/package-name-maps)的提议，它将提供一种机制，用于将模块名称映射到加载模块的路径。 这可能会得到浏览器和 Node 的支持，它们可以使用它来提供不同的路径，从而加载完全不同的模块，但使用相同的 API 。 通过这种方式，.wasm 模块本身可以指定一个（模块名称，函数名称）导入对，它可以在不同的环境(甚至Web)上运行。

有了这种机制，接下来要做的就是找出哪些函数对此有作用，以及它们的接口应该是什么。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-P-e1540221369730-500x187.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-01-P-e1540221369730.png)

目前还没有对此进行积极的研究。 但是现在很多讨论都朝这个方向发展。 它看起来很可能以某种形式发生。

这很好，因为解锁这些会让我们在浏览器之外实现一些用例。有了这些，我们可以加快步伐。

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-04-final-500x301.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-06-node-04-final.png)

那么，这些其他用例的一些例子是什么？

## CDN，无服务器和边缘计算

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-01-A-cloud-e1540221412524-500x278.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-01-A-cloud-e1540221412524.png)

例如CDN，无服务器和边缘计算。 在这些情况下，你将代码放在其他人的服务器上，并确保服务器得到维护，并且这些代码可以接近所有用户。

在这些情况下，为什么要使用 WebAssembly？ 在最近的一次会议上，有一场关于这一点的精彩演讲。

Fastly是一家提供 CDN 和边缘计算的公司。 他们的首席技术官 Tyler McMullen [是这样解释的](https://www.youtube.com/watch?v=FkM1L8-qcjU) ：

如果你查看一个进程是如何工作的，那么该进程中的代码就没有边界。 函数可以访问它们想要的那个进程中的任何内存，并且可以调用它们想要的任何其他函数。

当你在同一个进程中运行许多不同的人的服务时，这是一个问题。沙盒可能是解决这个问题的一种方法。然后这里会有一个规模问题。

例如，如果你使用像 Firefox 的 SpiderMonkey 或 Chrome 的 V8 这样的 JavaScript VM ，就像是使用一个沙盒，你可以将数百个实例放入一个进程中。但是随着请求数量的增长，Fastly 公司的每个进程中不只需要几百个实例，而是需要成千上万个。

Tyler 在他的演讲中做了更好的解释，你应该去看看他的演讲。但是这里问题的关键是 WebAssembly 提供了这个 Fastly 所需要的速度和规模，同时保证了安全性问题。

那么他们需要些什么来实现这个目标呢？

### 运行时

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-02-SS-e1540221444910-500x187.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-02-SS.png)

他们需要创建自己的运行时。 这意味着使用一个 WebAssembly 编译器(可以将WebAssembly 编译成机器代码的东西)，并将其与我前面提到的系统交互的函数相结合。

对于 WebAssembly 编译器，Fastly 使用 [Cranelift](https://github.com/CraneStation/cranelift) ，我们也在构建Firefox的编译器。我们将它设计得非常快，不会占用太多内存。

现在，对于与系统其余部分交互的功能，它们必须创建自己的函数，因为我们还没有可移植的接口。

所以现在创建自己的运行时是可能的，但这需要一些努力。并且不同公司要给出不同的解决方案。


如果我们不仅拥有可移植的接口，而且还拥有一个可以在所有这些公司和其他用例中使用的通用运行时呢？这肯定会加速发展。

这样，其他公司就可以使用那个运行时（就像他们今天做Node一样）而不是从头开始创建自己的。

### 我们进展如何？

那么我们的进展如何呢？

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-03-P-e1540221473263-500x187.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-03-P.png)

尽管目前还没有标准的运行时，但有一些运行时项目正在进行中。 其中包括构建在LLVM 之上的 [WAVM](https://github.com/WAVM/WAVM) 和 wasmjit。

另外，我们计划在 Cranelift 之上构建一个名为 wasmtime 的运行时。

一旦我们有一个通用的的运行时，就可以加速一系列不同用例的开发。 例如…

## 便携式CLI工具

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-05-A-cli-e1540221504824-500x274.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-05-A-cli.png)

WebAssembly 也可用于更传统的操作系统。 现在要清楚，我不是在内核中讨论（虽然[勇敢的人也在尝试这一点](https://github.com/nebulet/nebulet) ）但 WebAssembly 可以在 Ring 3 的用户模式下运行。

然后，你可以做一些事情，比如拥有可移植的 CLI 工具，可以跨所有不同类型的操作系统使用。

这与另一个用例非常接近......

## 物联网

[![img](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-05-A-iot-e1540221544774-500x277.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-05-A-iot.png)
物联网包括可穿戴技术和智能家电等设备。

这些设备通常是受资源限制的——它们没有太多的计算能力，也没有太多的内存。这种情况下，像 Cranelift 这样的编译器和 wasmtime 这样的运行时会很出色，因为它们效率高，内存低。在资源极度受限的情况下，WebAssembly 可以在将应用程序加载到设备之前完全编译成机器码。

还有一个事实是，这些不同的设备有很多，而且它们都略有不同。 WebAssembly的可移植性确实有助于这一点。

这就是 WebAssembly 未来的另一个地方。

## 结论

现在让我们缩小并查看这个技能树。

[![显示WebAssembly技能的技能树，将在帖子的过程中填写。](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-09-final-500x301.png)](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2018/10/01-07-runtime-09-final.png)

我在这篇文章的开头说过，人们对 WebAssembly 有一种误解——即 WebAssembly 的MVP 就是 WebAssembly 的最终版本。

我想你现在可以明白为什么这是一个误解。

是的，MVP 创造了很多机会。它使将许多桌面应用程序引入 web 成为可能。但我们仍然有很多用例可以解锁，从重型桌面应用程序，到小模块，到 JS 框架，再到浏览器之外的所有东西… Node.js，无服务器，区块链，可移植 CLI 工具，以及物联网。

所以我们今天拥有的 WebAssembly 并不是这个故事的结尾，它只是一个开始。

> 展开阅读：
> [WebAssembly](https://link.zhihu.com/?target=https%3A//webassembly.org/)
> [WebAssembly 中文网|Wasm 中文文档](https://link.zhihu.com/?target=http%3A//webassembly.org.cn/)
> [WebAssembly - MDN - Mozilla](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/WebAssembly)
> [WebAssembly: How and why – LogRocket](https://link.zhihu.com/?target=https%3A//blog.logrocket.com/webassembly-how-and-why-559b7f96cd71)
> [WebAssembly is here! – Unity Blog](https://link.zhihu.com/?target=https%3A//blogs.unity3d.com/2018/08/15/webassembly-is-here/)[WebAssembly is here! – Unity Blog](https://link.zhihu.com/?target=https%3A//blogs.unity3d.com/2018/08/15/webassembly-is-here/)[WebAssembly is here! – Unity Blog](https://link.zhihu.com/?target=https%3A//blogs.unity3d.com/2018/08/15/webassembly-is-here/)
> [Why WebAssembly is a game changer for the web — and a source of pride for Mozilla and Firefox](https://link.zhihu.com/?target=https%3A//medium.com/mozilla-tech/why-webassembly-is-a-game-changer-for-the-web-and-a-source-of-pride-for-mozilla-and-firefox-dda80e4c43cb)
> [WebAssembly 现状与实战](https://link.zhihu.com/?target=https%3A//www.ibm.com/developerworks/cn/web/wa-lo-webassembly-status-and-reality/index.html)

 