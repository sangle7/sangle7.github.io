---
title: 不为人知的JavaScript特性
date: 2019-11-30 10:50:44
tags: 
  - 翻译
---

原文作者：[Viral Shah](https://blog.usejournal.com/little-known-features-of-javascript-901665291387)

译者：Sangle

---

人们通常认为 JavaScript 是最简单的语言，也是最难掌握的语言。 我完全同意。 这是因为JavaScript是一种非常古老且非常灵活的语言。 它充满了神秘的语法和过时的功能。 到目前为止，我已经使用了 JavaScript 多年，时不时地，我仍然偶然发现一些我不知道的隐藏语法或技巧。

![](https://miro.medium.com/max/1000/0*wFHr5dO3zz74HyUU.jpg)

我试图列出一些鲜为人知的 JavaScript 功能。 尽管其中一些功能在严格模式下无效，但它们仍然是完全有效的 JavaScript 代码。 但是请注意，我不建议您开始使用所有这些功能。 尽管它们绝对酷，但如果您开始使用它们，很有可能会引起同事的怒火。

<!-- more -->

所有的源代码都可以在[这里](https://gist.github.com/viral-sh/98813f83f4afe9dce5a74e176f88724f)找到。 编码愉快！

> 注意：此处我说的特性不包括诸如变量提升，闭包，proxy，原型继承，async/await，generators 之类的东西。虽然这些功能大家可能不太理解，但它们仍然是众所周知的。

## Void 运算符

JavaScript 有一个不常用的 **void** 运算符，你可能在`void(0)`或`void 0`这样的场景下见过它。它只有一个单纯的目的-传入一个表达式，并返回 **undefined**。 传入'0'作为参数只是一种习惯。你不一定必须使用'0'，它可以是任何有效的表达式，例如`void <expression>`仍返回 **undefined**。

![void operator](https://miro.medium.com/max/964/1*xUnDmADcNEWApDU44EU3hQ.png)

> *为什么要创建一个特殊的关键字以返回 undefined 而不是单纯使用undefined？*
>
> *听起来有点多余，不是吗？*
>
> **🚩趣闻**
>
> *好吧，事实证明，在 ES5 之前，可以在大多数浏览器中为原始的 undefined 分配一个新值, 例如 
`undefined =“ abc”`。*
>
> *重新定义了 **undefined** ！*
>
> *在当时，使用 **void** 是确保始终返回 **undefined** 的一种方法。*

## 构造函数的括号是可选择的

是的，在调用构造函数时，我们在类名后添加的括号是完全可选的！ 😮（前提是您不需要将任何参数传递给构造函数）

以下两种代码样式均被视为有效的 JS 语法，并且将返回完全相同的结果！

![onstructor brackets are optional](https://miro.medium.com/max/1176/1*AIRCkc6i3wmKZGymJbnMcw.png)


## IIFE 的括号可以跳过
对于我来说，IIFE（立即调用功能表达式）的语法总是有点奇怪。

这么多括号都有什么作用？

事实证明，需要这些额外的括号只是告诉 JavaScript 解析器这段代码是***函数表达式***，而不是***函数***。 可以想象，知道了这一点，有很多方法可以跳过那些多余的括号，并且仍然可以写出**有效的 IIFE**。

![IIFE (without return)](https://miro.medium.com/max/1200/1*ssRSdg3vw9dbXHyJjL-3UQ.png)

**void** 运算符告诉解析器代码是函数表达式。 因此，我们可以跳过函数定义的括号。 你猜怎么着？ 我们可以使用任何**一元**运算符 *(void，+，！，-等)*，但这仍然有效！

太酷了！

但是，如果您是一个敏锐的观察者，您可能会想，

***一元** 运算符不会影响 **IIFE** 返回的结果吗？*

好吧，这会影响结果。 但是，好消息是，如果你将运行结果存储在某个变量中，那么就不需要多余的括号。

确实如此！

![IIFE with return](https://miro.medium.com/max/1178/1*aA6MfDN3p38PV2rXMXPB5w.png)

我们在 IIFE 中添加括号只是为了可读性。

*有关IIFE的更多信息，欢迎查看[Chandra Gundamaraju](https://medium.com/u/c9d1feae2937?source=post_page-----901665291387----------------------) 的这篇[很酷的文章](https://medium.com/@vvkchandra/essential-javascript-mastering-immediately-invoked-function-expressions-67791338ddc6)*

## With 表达式

你知道吗，JavaScript 有一个 **with** 语句块！ 实际上 **with** 是 JS 中的关键字。 编写 **with** 语句的语法如下：

```javascript
with (object)
   statement 
// for multiple statements add a block
with (object) {
   statement
   statement
   ...
}
```
在上面的栗子中，**with** 语句中的 `statement` 可以获得 `object` 对象和原型链上的所有属性

![with block example](https://miro.medium.com/max/1422/1*HgvRk6frw2rhJFs4OwvhaA.png)

> 🚩趣闻
>
> *with 语句块听起来非常酷，对吧？ 它甚至比[对象解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring)更好。*
>
> *好吧，这不是真的。*
>
> *通常不建议使用with语句，因为它已被弃用。 在严格模式下完全禁止这样做。 事实证明，使用with语句块会增加该语言的性能问题和安全性问题。*

## 函数构造器

函数表达式不是定义新函数的唯一方法，可以使用 `Function()` 构造函数以及 `new` 运算符动态地定义函数。

![Dynamic function with Function constructor](https://miro.medium.com/max/1398/1*q1yZNwt04aGew-ApvhZH0Q.png)

构造函数的最后一个参数，是函数的字符串化代码，其接受之前的其他参数为函数参数。

> 🚩趣闻
>
> ***Function** 构造函数是 JavaScript 中所有构造函数的父类。 甚至 **Object** 的构造函数都是 **Function** 构造函数。 **Function** 自己的构造函数也是 **Function** 本身。 因此，调用 `object.constructor.constructor...` 足够多的次数最终将返回 JavaScript 中任何对象上的 **Function** 构造函数。*

## 函数属性
众所周知，函数是JavaScript中的对象。 因此，没有人阻止我们向函数添加自定义属性。 在JS中，这是完全允许的。 但是，它很少使用。

那么我们什么时候想要这样做呢？

嗯，有一些很好的用例。 例如，

**可配置函数**

假设我们有一个叫做 *greet* 的函数。 我们希望我们的功能根据不同的语言环境打印不同的问候消息。 此语言环境也应该是可配置的。 我们可以在某个地方维护全局语言环境变量，也可以使用如下所示的函数属性来实现函数

![greet function with locale property](https://miro.medium.com/max/1284/1*HDgt3t2DGfXeFbblbTXkXw.png)

**带有静态变量的函数**

另一个类似的例子。 假设您要实现一个数字生成器，该数字生成器生成一系列有序数字。 通常，您将使用带有静态计数器变量的 **Class** 或 **IIFE** 来跟踪上一个值。 这样，我们限制了对计数器的访问，还避免了用额外的变量污染全局空间。
但是，如果我们想灵活地读取甚至修改计数器而又不污染全局空间怎么办？
好了，我们仍然可以创建一个带有计数器变量和一些其他读取它的方法的类； 或者我们可以很懒，只在函数上使用属性。

![generateNumber function with counter property](https://miro.medium.com/max/1318/1*kYy14RroTEJrn6eK8X1j9A.png)

---

*Phew!!  这是一个很长的文章，而目前我们大概读到了一半。 如果您想休息一下，现在将是个好时机。 如果不是，您就是勇敢的战士，我向您致敬。*

*让我们继续！*

---

## 参数属性

我敢肯定你们大多数人都知道函数内的 `arguments` 对象。 这是一个数组，类似于对象，可以在所有函数中使用。 它具有在调用时传递给函数的参数列表。 但是它还具有其他一些有趣的特性，
- **arguments.callee**：引用当前调用的函数
- **arguments.callee.caller**：引用已调用当前函数的函数

![callee & caller](https://miro.medium.com/max/1528/1*GolCj2Pgr2Hp-EumhLvPpw.png)

> *注意：尽管ES5禁止在**严格**模式下使用 `callee` 和 `caller`，但在许多已编译的库中仍然很常见。 因此，值得学习。*

## 标签模板字符串

除非您生活在上古时代，否则您应该听说过[*模板字符串*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)。 模板字符串是 ES6 提供的许多不错的功能之一。 但是，您知道***标签***模板字符串吗？

![Template literals](https://miro.medium.com/max/1216/1*d13JMz4wRyf2MmOh6-9XkQ.png)

*带有标签的模板字符串*可以通过向模板文字添加自定义标签，来更好地控制将模板文字解析为字符串。 Tag 只是一个解析器函数，它获取由字符串模板解释的所有字符串和值的数组。 标签函数应返回最终字符串。

在下面的示例中，我们的自定义标签 *highlight* 解释模板文字的值，并且还将解释后的值使用`<mark>`元素包装在结果字符串中以突出显示。

![highlight tagged template literal](https://miro.medium.com/max/1420/1*JsMvFe7D8XajNYHwzlvpCw.png)

> *许多库都发现了有趣的用例来利用此功能。 以下是一些很酷的例子，*
>
> - *[*styled-components*](https://github.com/styled-components/styled-components) for React*
>
> - *[*es2015-i18n-tag*](https://github.com/skolmer/es2015-i18n-tag) for translation & internationalization* 
>
> - *[*chalk*](https://github.com/chalk/chalk) for colorful logs*

## Getters & Setters

在大多数情况下，JavaScript对象很简单。 假设我们有一个对象 `user`，并尝试使用 `user.age` 访问其年龄属性，如果定义了年龄属性，我们将获得年龄属性的值；否则，将返回 **undefined**。 简单。

但是，并没有这么简单。 JavaScript 对象具有 **Getters** 和 **Setters** 的概念。 可以直接编写自定义 **Getter** 函数以返回所需的任何内容，而不是直接返回对象的值。 设置值也一样。

这使我们在获取或设置字段时拥有***虚拟字段，字段验证，副作用***等强大概念

![ES5 Getters & Setters](https://miro.medium.com/max/1288/1*2QkpXM8hkTIRtsUJPD5AEA.png)

Getters 和 Setters 并不是 ES5 的新增功能。 它们一直存在。 ES5只是在现有功能中添加了方便的语法。 要了解有关Getters＆Setters的更多信息，请参阅[这篇不错的文章](https://www.hongkiat.com/blog/getters-setters-javascript/)

> [Colors](https://github.com/Marak/colors.js) 是一个流行的 node.js 库，它是利用 **Getters** 的一个很好的例子。
>
> *该库[扩展了 String 类](https://github.com/Marak/colors.js/blob/master/lib/extendStringPrototype.js)，并在其上添加了一堆Getter 方法。 这使我们能够通过简单地[访问字符串上的属性](https://github.com/Marak/colors.js#usage)，将任何字符串转换为彩色，以便于输出日志。*

## 逗号运算符

JavaScript具有逗号运算符。 它允许我们在一行中编写多个用逗号分隔的表达式，并返回最后一个表达式的结果

```javascript
// syntax
let result = expression1, expression2,... expressionN
```
在这里，将对所有表达式进行求值，并将对结果变量赋值为 expressionN 返回的值。

您可能已经在 for 循环中使用了**逗号**运算符
```javascript
for (var a = 0, b = 10; a <= 10; a++, b--)
```
逗号运算符对于有时在一行中编写多个语句时会有所帮助
```javascript
function getNextValue() {
    return counter++, console.log(counter), counter
}
```
或编写简短的lambda函数
```javascript
const getSquare = x => (console.log (x), x * x)
```

## + 加号运算符

是否曾经想过将字符串快速转换为数字？

只需在字符串前面加上加号运算符即可。
加号运算符还适用于*负，八进制，十六进制，指数*值。 而且，它甚至可以将 *Date* 或 *Moment.js* 对象转换为时间戳！

![Plus operator](https://miro.medium.com/max/1224/1*9dbCIwXa7Kpj6xhbHHXl-g.png)


## !! Bang Bang 运算符

好的，从技术上讲，它不是独立的JavaScript运算符。 只是 JavaScript 否定运算符使用了两次。
但是 ***Bang Bang*** 听起来很酷！ *Bang Bang* 或 *Double Bang* 是将所有表达式转换为布尔值的巧妙技巧。
如果表达式是真实值，则返回 **true**, 否则返回 **false**。

![Bang Bang operator](https://miro.medium.com/max/1148/1*2ZqHFHfJvdKQpO4UnwxBIg.png)

## ~ 波浪运算符

面对现实吧-没人在乎波浪运算符。

我们什么时候才能使用它！

好吧，波浪运算符每天都有用例。
事实证明，当与数字一起使用时，波浪运算符有效
`~N => -(N + 1)`。 仅当`N == -1`时，此表达式的计算结果才为“0“
我们可以通过将〜放在`indexOf(...)`函数前面来利用此值，以对布尔值是否存在于String或Array中进行布尔检查。

![indexOf with Tilde operator](https://miro.medium.com/max/1278/1*lcfle5Zpfl2wU_rRsfDi4Q.png)

> *注意：ES6和ES7分别在String＆Array中添加了一个新的`.includes()` 方法。 无疑，这是一种比波浪号运算符更简洁的方法，来检查数组或字符串中是否存在指定项。*

## 标签语句

JavaScript具有`标签`语句的概念。 它允许我们在JavaScript中命名循环和块。 然后，我们可以在以后使用`break`或`continue`时使用这些标签来复用代码。

带标签的语句在嵌套循环中特别方便。 但是我们也可以使用它们将代码简单地组织成块或创建成小的代码块。

![labelled statements](https://miro.medium.com/max/1332/1*Fciyb5lUXJoqWxVWQoRHtw.png)

> *注意：与其他某些语言不同，JavaScript 没有 **goto** 语法。 因此，我们只能使用带有 **break** 和 **continue**的标签。*
