---
title: 理解 Virtual DOM
date: 2019-01-04 11:39:44
tags: 
  - 翻译
---
> 原文链接：https://bitsofco.de/understanding-the-virtual-dom/

我最近一直在写关于 [DOM](https://bitsofco.de/what-exactly-is-the-dom/) 的和 [shadow DOM](https://bitsofco.de/what-is-the-shadow-dom/) 以及它们之间区别的文章。 回顾一下，文档对象模型是 HTML 文档的基于对象的表示，提供操作该对象的接口。 shadow DOM 可以被认为是 DOM 的“精简”版本。 它也是 HTML 元素的基于对象的表示，但它不是完整的独立文档。 shadow DOM允许我们将 DOM 分成更小的封装单位，它们可以跨 HTML 文档使用。

您可能遇到的另一个类似术语是“ Virtual DOM ”。 尽管这个概念已存在多年，但它在 React 框架中的使用更受欢迎。 在本文中，我将详细介绍 Virtual DOM 的内容，它与DOM 的区别以及它的使用方式。

## 为什么需要 Virtual dom

为了理解 Virtual DOM 的概念出现的原因，让我们重新审视 DOM。 正如我所提到的，DOM 有两个部分：基于对象的 HTML 文档表示和操作该对象的 API。

例如，让我们将这个简单的 HTML 文档与无序列表和一个列表项一起使用。

```html
<!doctype html>
<html lang="en">
 <head></head>
 <body>
    <ul class="list">
        <li class="list__item">List item</li>
    </ul>
  </body>
</html>
```

<!-- more -->

这个 HTML 会转化为如下的 DOM 树：

![image-20181229091520994](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202019-01-03%20at%204.27.04%20PM.png)



假设我们想要将第一个列表项的内容修改为 “List item one”，并添加第二个列表项。 我们需要使用 DOM API 来查找我们想要更新的元素，创建新元素，添加属性和内容，然后最终更新 DOM 元素本身。

```javascript
const listItemOne = document.getElementsByClassName("list__item")[0];
listItemOne.textContent = "List item one";

const list = document.getElementsByClassName("list")[0];
const listItemTwo = document.createElement("li");
listItemTwo.classList.add("list__item");
listItemTwo.textContent = "List item two";
list.appendChild(listItemTwo);
```

### DOM不是为此诞生的......

当 [DOM 的第一个规范](https://www.w3.org/TR/REC-DOM-Level-1/)在1998年发布时，我们构建网页的方式和现在非常不同。 我们并不会像现在一样频繁的通过 DOM API 来创建和更新页面内容。

诸如 `document.getElementsByClassName()`之类的简单方法可以在小范围内使用，但如果我们每隔几秒更新页面上的多个元素，那么不断查询和更新 DOM 会变得非常昂贵。

更进一步，由于 API 的设置方式，在更新文档时，比起查找和更新特定元素所带来的昂贵的性能消耗，一次更新较大的范围通常会更简单。 回到我们的列表例子，我们使用新的元素整个替换会更合适。

```javascript
const list = document.getElementsByClassName("list")[0];
list.innerHTML = `
<li class="list__item">List item one</li>
<li class="list__item">List item two</li>
`;
```

在这个特定例子中，方法之间的性能差异可能是微不足道的。 但是，随着网页大小的增加，这些性能差异可能会非常明显。

### .....但 Virtual DOM是！

Virtual DOM 的诞生是为了解决需要以更高效的方式频繁更新DOM的这些问题。 与DOM或shadow  DOM不同，Virtual DOM不是官方规范，而是与 DOM 连接的新方法。

Virtual DOM 可以被认为是 DOM 的副本。 我们可以经常操作和更新此副本，而无需使用 DOM API。 完成对 Virtual DOM 的所有更新后，我们可以查看需要对 DOM 进行哪些特定更改，并以优化后的目标方式进行更改。



## Virtul DOM 长什么样？

”Virtul DOM“ 这个名称看起来很神秘，但事实上，它只是一个普通的 Javascript 对象。

让我们重温一下我们之前创建的DOM树：

![image-20181229091520994](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/Screen%20Shot%202019-01-03%20at%204.27.04%20PM.png)

这棵树也可以使用 JavaScript 对象来表示：

```javascript
const vdom = {
    tagName: "html",
    children: [
        { tagName: "head" },
        {
            tagName: "body",
            children: [
                {
                    tagName: "ul",
                    attributes: { "class": "list" },
                    children: [
                        {
                            tagName: "li",
                            attributes: { "class": "list__item" },
                            textContent: "List item"
                        } // end li
                    ]
                } // end ul
            ]
        } // end body
    ]
} // end html
```

我们可以将此对象视为我们的 virtual DOM。 与普通的 DOM 一样，它是我们的 HTML 文档的基于对象的表示。 但由于它是一个普通的 Javascript 对象，我们可以自由而频繁地操作它，而不需要操作实际的DOM。

更多的时候，我们不会将 Virutal DOM 应用于整个对象，而会在对象中使用 Virutal DOM 的小部分。 例如，我们可能会处理 `list` 组件，它会对应于我们的无序列表元素。

 ```javascript
const list = {
    tagName: "ul",
    attributes: { "class": "list" },
    children: [
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item"
        }
    ]
};
 ```

## 深入挖掘 Virtual DOM 

我们已经看到了 Virtual DOM 的样子，它是如何解决 DOM 的性能和可用性的问题呢？

正如我所提到的，我们可以使用 Virtual DOM 来选出需要在 DOM 上进行的特定更改，并单独进行这些特定更新。 让我们回到我们的无序列表示例，并使用 DOM API 进行相同的更改。

我们要做的第一件事是制作 Virtual DOM 的副本，其中包含我们想要进行的更改。 由于我们不需要使用 DOM API，因此我们实际上只需创建一个新对象。

```javascript
const copy = {
    tagName: "ul",
    attributes: { "class": "list" },
    children: [
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item one"
        },
        {
            tagName: "li",
            attributes: { "class": "list__item" },
            textContent: "List item two"
        }
    ]
};
```

此副本用于在初始的 Virtual DOM（在本例中为列表）和更新的 Virtual DOM 之间创建所谓的“差异”（diffs）。 差异可能看起来像这样：

```javascript
const diffs = [
    {
        newNode: { /* new version of list item one */ },
        oldNode: { /* original version of list item one */ },
        index: /* index of element in parent's list of child nodes */
    },
    {
        newNode: { /* list item two */ },
        index: { /* */ }
    }
]
```

此 diff 提供了有关如何更新实际 DOM 的说明。 一旦收集了所有差异，我们就可以批量更改 DOM，只进行所需的更新。

例如，我们可以循环遍历每个差异，并根据diff指定的内容添加新的子节点或更新旧的子节点。

```javascript
const domElement = document.getElementsByClassName("list")[0];

diffs.forEach((diff) => {

    const newElement = document.createElement(diff.newNode.tagName);
    /* Add attributes ... */
    
    if (diff.oldNode) {
        // If there is an old version, replace it with the new version
        domElement.replaceChild(diff.newNode, diff.index);
    } else {
        // If no old version exists, create a new node
        domElement.appendChild(diff.newNode);
    }
})
```

Note that this is a really simplified and stripped-back version of how a virtual DOM could work and there are lot of cases I didn't cover here.

请注意，这是一个介绍 Virtual DOM 如何工作 的例子，它时极度简化的，在这里有很多我没有涉及到的细节。

## Virtual DOM 和框架

更多的情况下，我们会通过框架来使用 Virtual DOM。

类似于 React 和 Vue 的框架使用了 Virtual DOM 来让减少 DOM 的更新优化性能。举个例子，`list` 组件在React 中可以写成以下的形式：

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const list = React.createElement("ul", { className: "list" },
    React.createElement("li", { className: "list__item" }, "List item")
);

ReactDOM.render(list, document.body);
```

如果我们想要更新列表，我们重新编写整个 list ，调用`ReactDOM.render()`并传入新列表

```jsx
const newList = React.createElement("ul", { className: "list" },
    React.createElement("li", { className: "list__item" }, "List item one"),
    React.createElement("li", { className: "list__item" }, "List item two");
);

setTimeout(() => ReactDOM.render(newList, document.body), 5000);
```

由于React 使用了 Virtual DOM，即使我们重新渲染了整个列表，也仅仅是发生了变化的部分会进行更新。通过开发者工具，我们可以看到具体变化的那些元素。具体操作可以参考下面这个视频：

https://res.cloudinary.com/ireaderinokun/video/upload/v1545416044/bitsofcode/react-virtual-dom.webm


## DOM vs virtual DOM

回顾一下，Virtual DOM 是一种工具，使我们能够以更简单，更高效的方式与DOM元素进行交互。 它将 DOM 表示为Javascript 对象，我们可以根据需要随时修改。 然后整理对该对象所做的更改，统一修改 DOM ，以降低修改 DOM 的频率。