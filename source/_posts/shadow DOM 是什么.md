---
title: shadow DOM 是什么？
date: 2018-12-23 10:14:44
tags: 
  - 翻译
---


> 原文链接： https://bitsofco.de/what-is-the-shadow-dom/

几周之前，我写了一篇关于[DOM究竟是什么](https://bitsofco.de/what-exactly-is-the-dom/)的文章。回顾一下，文档对象模型 (Document Object Model) 是对HTML 文档进行解析获得的表现形式。 浏览器使用它来确定页面上要呈现的内容，并通过 Javascript 来修改页面的内容，结构或样式。

举个例子，我们看看下面这个 HTML 文件：

```html
<!doctype html>
<html lang="en">
 <head>
   <title>My first web page</title>
  </head>
 <body>
    <h1>Hello, world!</h1>
    <p>How are you?</p>
  </body>
</html>
```

这个 HTML 文件会转化为如下的 DOM 树：

![dom tree](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202018-12-24%20at%2010.01.05%20AM.png)

在过去几年中，你可能听说过“Shadow DOM”和“Virtual DOM”等术语。 虽然它们与 DOM 有关，但事实上，它们上指的是截然不同的概念。 在本文中，我将详细介绍 shadow DOM 以及它与 DOM 的区别。 在以后的文章中，我将会介绍 virtual DOM。

<!-- more -->

## 一切都是全局的 👍🏾! 一切都是全局的 👎🏾

HTML 文档中的所有元素和样式以及 DOM 都位于一个全局范围内。 使用`document.querySelector()`方法可以访问页面上的所有元素，无论该元素在文档中的任何地方。 同样，应用于文档的 CSS 可以选择任何元素，无论元素在何处。

当我们想要将样式应用于整个文档时，这种特性非常有用。它允许我们能够仅仅使用一行代码就设置页面上的所有元素的盒模型。

```css
* { box-sizing: border-box }
```

另一方面，有些时候元素需要完全封装，我们不希望它受到全局样式的影响。 一个很好的例子是第三方小部件，例如Twitter的“关注”按钮。 这个组件看起来像是这样：

![](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202018-12-24%20at%2010.01.16%20AM.png)

假设你启用了Javascript，在控制台审查元素时，你会注意到该按钮是一个<iframe>元素，**它使用您实际看到的样式按钮加载一个小文档。**

![Follow-button-widget-iframe](https://bitsofco.de/content/images/2018/12/Follow-button-widget-iframe.png)

通过这种方式，Twitter 可以确保这个关注组件的样式将不受到其所在文档中的任何 CSS 影响，这也是唯一的方式。 虽然有一些方法尝试使用级联来获得相同的结果，但都并不理想，它们不能达到和 <iframe> 相同的效果。

事实上， <iframe> 并不是为为了解决这个问题而诞生的。Shadow DOM 的出现使我们可以在 Web 平台上进行样式隔离和组件化，而不必依赖像 <iframe> 这样的工具，

## A DOM within a DOM

你可以认为 shadow DOM 类似于 “A DOM within a DOM”，它是一个独立的 DOM 树，具有自己的元素和样式，与普通的 DOM 完全隔离。

虽然 shadow DOM 近年来才提供给 web 开发者使用，但浏览器 （user agent？） 已经使用了 shadow DOM 很多年，它们用来创建和设置复杂组件（如表单元素）的样式。我们来看一下范围输入元素。 首先，我们在页面上添加一个范围输入的元素：

```html
<input type="range">
```

这一个元素会生成以下组件：

![range-input](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202018-12-24%20at%2010.01.44%20AM.png)

如果我们打开元素审查，我们可以看到这个<input>元素实际上由几个较小的<div>元素组成，这几个元素用来控制轨道和滑块。

![Range input shadow dom](https://bitsofco.de/content/images/2018/12/Range-input-shadow-dom.png)

这是使用 shadow DOM 实现的。 暴露给宿主 HTML 的元素只有简单的<input>，但在其下面有与组件相关的元素和样式，这些元素和样式并不在 DOM 的全局范围中。

## shadow DOM 的工作原理？

为了说明 shadow DOM的工作原理，让我们使用 shadow DOM 替代 <iframe> 来重新创建 Twitter 的“关注”按钮。

首先，我们从 **shadow host** 开始。它是我们想要将新 shadow DOM 附加到原始DOM中的常规HTML元素。 对于像关注按钮这样的组件，它还可以包含我们希望在页面上未启用 Javascript 或不支持shadow DOM时显示的回退元素。

```html
<span class="shadow-host">
  <a href="https://twitter.com/ireaderinokun">
     Follow @ireaderinokun
  </a>
</span>
```

请注意，我们不使用 <a> 元素作为shadow host，因为某些元素不能作为shadow host。

我们使用`attachShadow()`方法将shadow DOM 附加到我们的 shadow host 上

```js
const shadowEl = document.querySelector(".shadow-host");
const shadow = shadowEl.attachShadow({mode: 'open'});
```

This will create an empty **shadow root** as a child of our shadow host. The shadow root is the start of a new shadow DOM in the way that the `<html>` element is the start of the original DOM. We can see our shadow root in the devtools inspector by the **#shadow-root**.

这会在我们的shadow root中创建一个空的**shadow root** 作为子元素，shadow root 是一个shadow DOM被添加到原始DOM的html元素标志，我们可以通过#shadow-root在devtools检查器中看到我们的shadow root。

![Empty shadow root](https://bitsofco.de/content/images/2018/12/Empty--shadow-root.png)

尽管在 shadow root 下的常规HTML元素在 devtools 中是可见的，但是在页面上，它们被 shadow root 接管，不再可见。

接下来，我们要创建内容以形成新的 **shadow tree**。shadow tree 就像一个DOM树，区别在于它是针对阴影 DOM 的而不是常规 DOM 。 要创建我们的关注按钮，我们所需要的只是一个新的`<a>`元素，同时带有一个图标。

```js
const link = document.createElement("a");
link.href = shadowEl.querySelector("a").href;
link.innerHTML = `
    <span aria-label="Twitter icon"></span> 
    ${shadowEl.querySelector("a").textContent}
`;
```

我们将这个新元素添加到我们的 shadow DOM 中，就和使用`appendChild()`方法添加子元素一样。

```js
shadow.appendChild(link);
```

这时，我们的元素看起来像是这样：

![Plain text of "Follow-@ireaderinokun"](https://bitsofco.de/content/images/2018/12/Plain-text-of--22Follow-@ireaderinokun-22.png)

最后，我们可以通过创建<style>元素并将其附加到 shadow root 来添加一些样式。

```css
const styles = document.createElement("style");
styles.textContent = `
a, span {
  vertical-align: top;
  display: inline-block;
  box-sizing: border-box;
}

a {
    height: 20px;
    padding: 1px 8px 1px 6px;
    background-color: #1b95e0;
    color: #fff;
    border-radius: 3px;
    font-weight: 500;
    font-size: 11px;
    font-family:'Helvetica Neue', Arial, sans-serif;
    line-height: 18px;
    text-decoration: none;   
}

a:hover {  background-color: #0c7abf; }

span {
    position: relative;
    top: 2px;
    width: 14px;
    height: 14px;
    margin-right: 3px;
    background: transparent 0 0 no-repeat;
    background-image: url(data:image/svg+xml,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20viewBox%3D%220%200%2072%2072%22%3E%3Cpath%20fill%3D%22none%22%20d%3D%22M0%200h72v72H0z%22%2F%3E%3Cpath%20class%3D%22icon%22%20fill%3D%22%23fff%22%20d%3D%22M68.812%2015.14c-2.348%201.04-4.87%201.744-7.52%202.06%202.704-1.62%204.78-4.186%205.757-7.243-2.53%201.5-5.33%202.592-8.314%203.176C56.35%2010.59%2052.948%209%2049.182%209c-7.23%200-13.092%205.86-13.092%2013.093%200%201.026.118%202.02.338%202.98C25.543%2024.527%2015.9%2019.318%209.44%2011.396c-1.125%201.936-1.77%204.184-1.77%206.58%200%204.543%202.312%208.552%205.824%2010.9-2.146-.07-4.165-.658-5.93-1.64-.002.056-.002.11-.002.163%200%206.345%204.513%2011.638%2010.504%2012.84-1.1.298-2.256.457-3.45.457-.845%200-1.666-.078-2.464-.23%201.667%205.2%206.5%208.985%2012.23%209.09-4.482%203.51-10.13%205.605-16.26%205.605-1.055%200-2.096-.06-3.122-.184%205.794%203.717%2012.676%205.882%2020.067%205.882%2024.083%200%2037.25-19.95%2037.25-37.25%200-.565-.013-1.133-.038-1.693%202.558-1.847%204.778-4.15%206.532-6.774z%22%2F%3E%3C%2Fsvg%3E);
}
`;

shadow.appendChild(styles);
```

这是我们最终生成的元素：

![image-20181224092104072](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202018-12-24%20at%2010.01.16%20AM.png)



## The DOM vs the shadow DOM

在某些方面，shadow DOM 是 DOM 的“精简”版本。 与 DOM 一样，它使用 HTML 元素，用于页面呈现内容，并且允许对这些元素进行修改。 但与DOM不同，shadow DOM 不基于独立文档。 正如其名称所示，shadow DOM 是附加到常规 DOM 中的元素。 没有 DOM，阴影 DOM 就不存在了。