---
title: 在 JavaScript 中克隆数组
date: 2019-05-29 11:20:44
tags: 
  - JavaScript
---

> 原文：https://stackabuse.com/clone-arrays-in-javascript/

在我之前的一篇文章中，我介绍了如何在JavaScript中复制对象。 复制对象是一项相当复杂的事，考虑到你必须要复制对象中可能存在的所有其他数据类型。 但是，就像上一篇文章一样，有很多方法可以完成数组的复制，其中一些我将在本文中介绍。

但首先要强调的是，关于复制数组的速度。虽然这对大部分应用程序来说都无关紧要，但如果复制大型数组是代码中的常见操作，或者在你的项目中速度确实很重要，则这一点是需要考虑的。对于下面的一些方法，我记录了其相对于其他方法的速度，这些数据来自于基准测试。

## 浅拷贝

在第一种场景下，我们假设要复制的数组仅包含原始数据类型。 也就是说，数组只包含数字，布尔值，字符串等。这样我们可以更专注于从一个数组到另一个数组的数据传输，而不是我们如何处理复制数组的实际内容，这部分可以请参阅下面的“深拷贝”部分。

有许多方法可以复制数组，其中一些方法包括：

- `push`
- Spread
- `slice`
- `Array.from`
- `_.clone`

<!-- more -->

扩展运算符和`slice`方法是复制浅层数组的最快方法，但请记住，这取决于底层运行环境，因此可能不是普遍适用的。

### Push

这可能是最显而易见的解决方案，循环遍历原始数组并使用新数组的`push()`方法将元素从一个数组添加到另一个数组：

```javascript
let oldArr = [3, 1, 5, 2, 9];  
let newArr = [];  
for (let i=0; i < oldArr.length; i++) {  
    newArr.push(oldArr[i]);
}
```
我们只需要循环遍历要复制的数组，并将每个元素push到新数组。


### Spread

这种方法使用扩展运算符，该运算符在ES6中定义，可在大多数最新的浏览器中使用。

它的使用方式如下：

```javascript
let oldArr = [3, 1, 5, 2, 9];  
let newArr = [...oldArr];  
```
如果我打算使用原生解决方案而不使用第三方库，那么这通常是我喜欢的解决方案，这要归功于其简洁明了的语法。

一个重要的注意事项是，此复制仅适用于前拷贝，因此如果你需要执行任何深拷贝，则不应使用它。

### Slice

`slice()`方法通常用于返回由开始和结束参数指定的数组的一部分。 值得注意的是，如果没有传递参数，则返回整个数组的副本：

```javascript
let oldArr = [3, 1, 5, 2, 9];  
let newArr = oldArr.slice();  
```

在许多JavaScript运行环境中，这是复制数组的最快方法。

### Array.from

`Array.from` 方法用于创建传递给它的任何可迭代的浅层副本，它还将可选的映射函数作为第二个参数。 所以它可以用来创建一个字符串，集合，映射，当然还有其他数组：

```javascript
let oldArr = [3, 1, 5, 2, 9];  
let newArr = Array.from(oldArr);  
```

### Lodash Clone

如果你阅读有关复制对象的文章，你可能很熟悉 Lodash 的`clone()`和`cloneDeep()`方法。 这些方法完全符合我们的期望 - 传递给它的任何对象都将被复制和返回。

`_.cloneDeep`（下面进一步描述）的不同之处在于它不仅是顶层的克隆，而会以递归方式复制它在任何级别遇到的所有对象。

鉴于此，我们也可以使用它来复制数组：

```javascript
let oldArr = [3,1,5,2,9];
let newArr = _.clone（oldArr）;
```

与其他方法相比，`_.clone`表现得非常好，所以如果你已经在应用程序中使用了这个库，那么这是一个简单的解决方案。

## 深拷贝

需要指出的一点是，上面描述的所有方法只执行数组的浅拷贝。 因此，如果你有一个对象数组，例如，将复制实际数组，但底层对象将通过引用传递给新数组。

为了演示这个问题，让我们看一个例子：

```javascript
let oldArr = [{foo: 'bar'}, {baz: 'qux'}];  
let newArr = [...oldArr];  
console.log(newArr === oldArr);  
console.log(newArr[0] === oldArr[0]);  
```
```
false
true
```
在这里你可以看到，虽然实际的数组是新的，但其中的对象却不是。 对于某些应用程序，这可能是一个大问题。 如果这适用于你，那么这里有一些其他方法。

### Lodash Clone Deep

Lodash的`_.cloneDeep`方法与`_.clone（）`完全相同，只是它以递归方式克隆传递它的数组（或对象）中的所有内容。 使用与上面相同的示例，我们可以看到使用`_.cloneDeep()`将为我们提供新数组和复制的数组元素：

```javascript
const _ = require('lodash');

let oldArr = [{foo: 'bar'}, {baz: 'qux'}];  
let newArr = _.cloneDeep(oldArr);  
console.log(newArr === oldArr);  
console.log(newArr[0] === oldArr[0]);  
```
```
false
false
```

### JSON Methods
JavaScript确实提供了一些方便的JSON方法，它们处理将大多数JS数据类型转换为字符串，然后将有效的JSON字符串转换为JS对象。 方法使用如下：

```javascript
let oldArr = [{foo: 'bar'}, {baz: 'qux'}];  
let arrStr = JSON.stringify(oldArr);  
console.log(arrStr);

let newArr = JSON.parse(arrStr);  
console.log(newArr);

console.log(newArr === oldArr);  
console.log(newArr[0] === oldArr[0]);  
```
```
'[{"foo":"bar"},{"baz":"qux"}]'  
[ { foo: 'bar' }, { baz: 'qux' } ]
false  
false  
```
此方法很有效，并且不需要任何第三方库。 但是，有两个主要问题：

- 数据必须可通过JSON进行序列化和反序列化
- 以这种方式使用JSON方法比其他解决方案慢得多

因此，如果你的数据无法序列化为JSON，或者速度对你的应用程序很重要，那么这对你来说可能不是一个好的解决方案。

## 总结
在本文中，我介绍了一些可以在JavaScript中复制数组的方法，包括使用本机代码以及Lodash中有用的第三方库。 我们还研究了深度克隆数组的问题以及解决它的解决方案。

是否有一种最适合你的方法？ 请在留言中让我们知道你的想法。