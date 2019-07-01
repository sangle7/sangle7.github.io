---
title: 使用 Web Workers 优化 JavaScript 应用程序性能
date: 2019-06-29 18:25:44
tags: 
  - JavaScript
---
> 原文链接：https://www.twilio.com/blog/optimize-javascript-application-performance-web-workers


从简单的脚本语言到成为 Web 标准编程语言，JavaScript 已经走过了漫长的道路。时至今日，它已经被广泛用于构建服务器端应用程序，移动应用程序，桌面应用程序甚至数据库。

尽管 JavaScript 是用于在Web上构建复杂且引人入胜的软件的优秀语言，但由于JavaScript语言的性质，可能会将性能低效引入这些应用程序。

在本文中，您将学习如何使用 Web worker 修复 Web 应用程序中长时间运行的脚本导致的性能问题。 [Web worker](https://en.wikipedia.org/wiki/Web_worker) 是一个在后台运行的 JavaScript 脚本，与从同一 Web 页面执行的用户界面脚本无关。

<!-- more -->


## 先决条件

首先，你需要一个开发服务器。 如果你尚未安装，则可以选择[适用于 Google Chrome 的 Chrome 扩展程序](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb?hl=en)。

本文章假设读者拥有 HTML，CSS 和 JavaScript 的基本知识。

本文章的[项目实例代码](https://github.com/9jaswag/web-performance-with-workers)可在GitHub上找到。



## JavaScript 主线程

JavaScript 是单线程的，这意味着在同一时间只有一段代码能够运行。像是UI更新，用户交互，图片缩放之类的任务需要被放进一个任务队列，并使用浏览器的 JavaScript 引擎依次执行。

这个单线程的设计模式为性能带来的最大问题就是阻塞。当主线程执行一个需要非常长时间的任务时，阻塞就会发生，阻塞会影响其他所有任务的执行，会导致web程序执行缓慢或是卡顿，这对于用户体验来说是非常糟糕的。

为了解决阻塞的问题，JavaScript 提供了一个 API 来在独立于主线程之外的后台运行 JavaScript 脚本。这就是 [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)。



## Web Workers

根据 MDN 的文档：“ Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面。”

Web Workers 允许你生成新线程，并将一些工作放在这些线程中执行以获得高性能。 在这种情况下，我们通常会把需要长时间执行的任务交给 Worker，从而保证主线程可以在不被阻塞的情况下运行。



## 生成 Web Worker

一个 Web Worker 是一个 JavaScript 文件。要生成一个 Web Worker，创建一个含有你想要在 worker 线程中执行的代码，并在主文档中使用 Worker 构造器这样引入：

```javascript
const worker = new Worker('worker.js');
```

上面的代码片段生成了一个变量名为 `worker` 的 Worker，通过这种方式，这个 worker 已经可以发送和接收主线程的数据。



## 与 Web Worker 通信


Web Workers 和主线程通过消息系统相互发送数据。此消息可以是任何值，例如字符串，数组，对象，甚至是布尔值。

Web Worker API 提供了一个 `postMessge()` 方法，用于向 Worker 发送消息和从 Worker 发送消息，以及用于接收和响应消息的 `onmessage `事件处理程序。

要向 worker 发送消息或从 worker 发送消息，请在 worker 对象上调用 `postMessage()` 方法：

```javascript
// send data from a JavaScript file to a worker
worker.postMessage(data);

// send data from a worker to a JavaScript file
self.postMessage(data);
```

要从 worker 线程或主线程接收数据，则需要为消息事件创建事件监听器，并通过`data`访问它：

```javascript
// receive data from a JavaScript file
self.onmessage = (event) => {
  console.log(event.data);
}

// receive data from a worker
worker.onmessage = (event) => {
  console.log(event.data);
}
```

值得注意的是，在 worker 线程和主线程之间传递的数据将被复制而不会被共享。



## 终止 Web Worker

创建 Web Worker 会在用户的计算机上生成实际线程，从而消耗系统资源。因此，一个比较好的做法是在 worker 执行完毕后终止 worker。可以通过调用 worker 上的 `terminate()` 方法终止 worker。无论是否正在执行任务，这都会立即终止 worker。worker 也可以在它自己的线程内被终止。为此，可以在 worker 中调用`close()`方法：

```javascript
// close a worker from the main script
worker.terminate();

// close a worker from itself
self.close();
```



## Web Workers 的限制

 Web Workers API是一个非常强大的工具，但它有一些限制：


>   - Worker 不能直接操作 DOM，并且对 window 对象的方法和属性的访问权限有限。
>   - 无法直接从文件系统运行 worker。它只能通过服务器运行。



## 创建示例程序

我们将创建一个示例程序来演示运行脚本对 Web 应用程序性能的影响。确保在继续之前已在 Chrome 中安装了 Web Server for Chrome 扩展程序。 

在您的计算机上创建一个新文件夹 web_workers，并在 web_workers 文件夹中创建一个 index.html 文件。将以下代码添加到文件中：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.2/css/all.css"
integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous">
  <link rel="stylesheet" href="styles.css">
  <script src="index.js"></script>
  <title>Flight</title>
</head>

<body>
  <i class="fas fa-space-shuttle fa-4x shuttle red" id="shuttle1"></i>
  <i class="fas fa-space-shuttle fa-4x shuttle blue" id="shuttle2"></i>
  <i class="fas fa-space-shuttle fa-4x shuttle green" id="shuttle3"></i>

  <button id="start" onclick="move()">Start</button>
  <button id="calculate" onclick="calculate()">Run Calculation</button>
</body>

</html>
```

上面的代码包含一个带有三个 `<i>` 标签的基本 HTML 文档，每个标签都有一个航天飞机图标，还有两个按钮。单击第一个按钮时，航天飞机图标应从左向右移动。单击第二个按钮会运行CPU大量计算。 



创建一个 styles.css 文件并添加补充样式：

```css
.shuttle {
  display: block;
  margin: 3rem 0;
  position: relative;
}

.red {
  color: red;
}

.blue {
  color: blue;
}

.green {
  color: green;
}

button {
  background-color: #e83c6c;
  border: none;
  border-radius: 0.3rem;
  color: white;
  padding: 0.7rem 1.5rem;
  font-weight: bold;
  cursor: pointer;
}
```

要查看演示应用程序，请通过单击Chrome书签栏中的“应用”快捷方式或导航到 chrome://apps 来打开Web Server for Chrome。 



单击“选择文件夹”按钮，然后选择计算机上任何位置的 web_workers 文件夹。单击切换按钮以启动服务器并访问 Web Server for Chrome 界面中显示的 Web 服务器 URL。您应该看到类似于下图的内容：

![example screenshot for app](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/amCy3k5uZGaofY2sxjOELE1UaO7SUw5MCpmW-qNAD9jy3.width-1600.png)

要使按钮起作用，请在 web_workers 文件夹中创建 index.js 文件，并将以下代码添加到其中：

```javascript
const fibonacci = (num) => {
  if (num <= 1) return 1;

  return fibonacci(num - 1) + fibonacci(num - 2);
}

const calculate = () => {
  const num = 40;

  console.log(fibonacci(num));
  return fibonacci(num);
}

const move = () => {
  for (let i = 0; i < 3; i++) {
    const shuttleID = `shuttle${i + 1}`;
    let shuttle = document.getElementById(shuttleID);

    let position = 0;
    setInterval(() => {
      if (position > window.innerWidth / 1.2) {
        position = 0;
      } else {
        position++;
        shuttle.style.left = position + "px";
      }
    }, 5);
  }
};
```

上面的代码有是三个函数；`move`函数将页面上的图像每 5 毫秒向前移动 1px，`calculate` 函数返回 斐波那契序列中的第40个数字。以及一个 `fibonacci`函数，它保存用于计算所提供数字的索引值的逻辑斐波那契序列使用递归。计算斐波那契序列中的第 40 个数字是资源密集型的，它需要几秒钟才能运行完毕。

刷新浏览器中的示例程序并点击 **Start** 来移动这些图片。在它们移动的任何时间，点击 **Run calculation **来进行斐波那契计算。你会观察到这些图片的移动静止了几秒，这是一个长时间运行的脚本如何影响 Web 应用程序性能的直观展示。

为了探究动画冻结的原因，重新加载浏览器标签，打开开发者工具（F12 或 Ctrl + Shift + I），切换到 Performance 标签页。点击录制按钮（Ctrl+E）来启动 JavaScript profilling。然后点击页面上的 **Start** 按钮，随后点击  **Run calculation** 按钮。

在动画冻结几秒后，点击开发者工具中的结束录制，你会获得一张和下图相似的结果：

![Performance metrics tab](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/IMPxdcDESTjnZMkQWgFFf85fpHK0BTjIE1VZ_Ul6HNBXn.width-1600.png)

上图中的高亮部分显示了主线程上的活动， 右上角显示一个红色三角形的是点击事件。这个点击事件导致了 index.js 文件中第21行的函数调用，该文件又调用了几次 `fibonacci` 函数。

事件上的红色三角形是一个警告，表示该事件与性能问题有关。 这表明`fibonacci`函数直接导致页面上的动画冻结。

## 通过 Web Workers 优化性能

为了确保演示应用程序中的动画穿梭不受斐波那契计算的影响，斐波纳契计算的递归逻辑需要从主线程移出。

 在 web_workers 文件夹中创建一个文件 worker.js，并从 index.js 文件中移动 `fibonacci` 函数：

```javascript
const fibonacci = (num) => {
  if (num <= 1) return 1;

  return fibonacci(num - 1) + fibonacci(num - 2);
};
```

现在计算逻辑已移动到 worker，只要单击计算按钮，就要调用它。在 index.js 文件中，通过将`fibonacci` 函数替换为以下语句，创建一个新的 worker 实例并将其链接到 worker.js 文件：

```javascript
let worker = new Worker("./worker.js");
```


更新index.js文件中的calculate函数，将我们想要计算斐波那契序列中索引值的数字发送给 worker：

```javascript
const calculate = () => {
  const num = 40;
  worker.postMessage(num);
};
```

每当调用计算函数时，数字 40 被发送给 worker 以计算斐波纳契数列中的第 40 个数字。要在 worker 中获取此数字，请使用以下代码在 worker.js 文件的顶部添加 onmessage 事件监听器。

```javascript
worker.onmessage = event => {
  const num = event.data;
  console.log(num);
};
```

在浏览器中重新加载应用程序，启动动画，然后点击 **Run calculation** 按钮。您将观察到斐波纳契序列计算的结果仍然记录在浏览器控制台中，但这不会影响页面上图像的移动。 

要确定 web worker 的性能影响，打开开发者工具并选择 “Performance” 选项卡。

重复上述步骤以开始性能分析，运行动画并执行斐波纳契计算。您应该看到类似于下面显示的结果：

![Performance after optimization](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/9r1HJhYKcglQv2zilM6I2kXxmxACNFOqZjHtZ9loDCljA.width-1600.png)

从上图中可以看出，第一个明显的变化是主线程旁边存在一个 worker 线程。 worker 线程在 worker.js 文件中显示一个带有 `onmessage` 事件的函数调用，该事件又调用 `fibonacci` 函数多次。 这表明斐波那契计算不再发生在主线程上，因此改善了航天飞机动画的性能。



## 总结

在这篇文章中，您了解了脚本运行时长对 Web 性能的影响以及如何使用 Web Workers API 修复这些性能问题。 同时，您还了解了如何使用 Google Chrome 开发者工具来分析 JavaScript 应用程序的性能，从而可以快速识别哪些代码是性能问题的瓶颈，并将它们移动到 web worker 中来避免性能问题。



## 其他资源

[MDN 的 Web Worker API 文档](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) 是一个很好的了解 Web Workers 的资源。

W3Schools 的 [Web Workers](https://www.w3schools.com/html/html5_webworkers.asp) 文档同样是一个很好的资源。

你可以在 [Github](https://github.com/9jaswag/web-performance-with-workers) 上看到这篇文章中的示例代码