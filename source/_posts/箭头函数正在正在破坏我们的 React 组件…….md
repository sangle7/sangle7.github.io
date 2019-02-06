---
title: 箭头函数正在正在破坏我们的 React 组件……
date: 2019-02-05 16:50:44
tags: 
  - 翻译
  - react
---

原文作者：[Karthik Kalyanaraman](http://link.zhihu.com/?target=https%3A//blog.usejournal.com/%40karthikkalyanaraman)

译者：Sangle

> 译者的话：JS 类中的箭头函数只是语法糖，在编译之后，它将出现在构造函数中，在实例化时创建并分配，这就是为什么我们没有在原型中看到这些方法。在使用箭头函数（而不是手动 bind）的情况下，每次使用组件时都会创建一个全新的函数，这会对应用程序的性能和内存使用产生负面影响。

![img](https://pic2.zhimg.com/80/v2-233397a5f458259fd08088d95cd848c1_hd.jpg)

你是否有过疑问，为什么我们要在 React Components 的构造函数中将类方法绑定到 'this' 上？当你的代码越来越复杂，有很多回调函数时，构造函数看起来就像这样丑陋和繁琐：

<!-- more -->

```
class Foo extends Component {
  constructor(props) {
    this.cb1 = this.cb1.bind(this)
    this.cb2 = this.cb2.bind(this)
    this.cb3 = this.cb3.bind(this)
  }
  cb1(){}
  cb2(){}
  cb3(){}
  render() {
    <Button onClick={this.cb1}>
     click me
    </Button>
  }
}
```

最近，我的一位来自开源社区的朋友建议使用箭头函数。使用箭头函数，就不必像这样在构造函数中绑定 'this'：

```
class Foo extends Component {
  constructor(props) {
  }
  cb1 = () => {}
  cb2 = () => {}
  cb3 = () => {}
  render() {
    <Button onClick={this.cb1}>
     click me
    </Button>
  }
}
```

这看起来很棒！使用箭头函数使我们的代码看起来比前面的例子更清洁。 我决定深入了解箭头函数，看看它真正做了什么以了解它的用法。

在理解箭头函数的工作原理之前，让我们回顾一下基础知识。'bind' 是什么，我们为什么需要进行绑定？

> **bind()**方法创建一个新的函数，在调用时设置`this`关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。[[MDN](http://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)]

让我们看看下面的例子：

```
function foo() {
  console.log(this.name)
}
var man = {"name": "jack"}
foo() // undefined
// bind man to foo
foo = foo.bind(man)
foo() // jack
```

所以，在将 `man` 绑定到 `foo` 之前，`foo` 是一个常规函数，没有引用 'name'，因此，当你调用 `this.name`时，它不会打印任何内容。 但是，当我们将 `man`绑定到 `foo` 对象时，`foo` 现在有一个 'this' 引用（现在是'man'），它会从 `man` 对象中打印出 'name'。

现在，我们已经了解什么是 bind 了，让我们更进一步，了解在特殊情况下会发生什么，

```
function foo() {
  console.log(this.name)
}
var man = {"name": "jack"}
foo() // undefined
// Attach foo to man as a property
man.foo = foo
man.foo() // jack
// Copy man.foo to a new var bar
var bar = man.foo
bar() // undefined (WTF?)
```

当我们将 `foo` 附加到 `man` 并将在 `man` 上调用时，可以看到，`foo` 可以获取 `man` 的名字并正确地打印了 'jack'。 但当我们创建一个新的变量 `bar` 并将 `man.foo` 复制到 `bar` 时会发生什么？ 我们看到突然间，`man.foo` 失去了 'this' 。 因为从 `bar` 中获取的 'this' 并不是 `man`，而是全局的 window 对象，在 window 下，我们没有定义 'name' 字段。 如何解决这个问题？没错， 我们可以使用 `bind` ！

```
foo = foo.bind(man)
man.foo = foo
var bar = man.foo
bar() // jack
```

现在它生效了。但是它在类中会发生什么呢？请记住，JavaScript 中的类只是构造函数的语法糖，这意味着，类中的表现与这个完全相同。

但是，为什么我会将类方法复制到一个变量上使用？听起来很没有道理，但这正是我们在创建回调时发生的……

> **“回调”**

还记得按钮组件的onClick属性吗？

```
class Foo extends React.Component {
  constructor() {
    this.clickhandler = this.clickhandler.bind(this)
  }
  clickhandler() {
    console.log("you clicked me!!")
  }
  render() {
    return( 
    <div>
      <button onClick={this.clickhandler}> // => CALLBACK
        Click me
      </button>
    </div>)
  }
}
```

噢！ 花了一段时间我们终于把他们联系了起来。现在，这一切都说得通了。 好的！ 那么，箭头函数是什么呢？ 为什么通过箭头函数，我们不用绑定就可以让函数正常工作？

让我们回到最开始

```
var name = "rose"
var foo = () => { console.log(this.name) }
foo() // rose
var man = {"name": "jack"}
man.foo = foo
man.foo() // rose
foo.bind(man)
man.foo() // rose
```

Why am I not able to bind or attach man’s ‘name’ property to foo at all?

为什么我无法将 man 的 name 属性绑定或附加到 foo 上？

> “这是因为，你不能”重新绑定“箭头功能。 它将始终使用定义它的上下文进行调用。“

这也意味着，作为回调的箭头功能将正常工作。

因为，回调箭头函数，clickhandler 是在上下文中永久定义的，这是 Foo 类的上下文。 但是，当您将其分配给变量或者，换句话说，将其用作回调时，常规函数会丢失上下文。

```
class Foo extends React.Component {
  constructor() {
  }
  clickhandler = () => {
    console.log("you clicked me!!")
  }
  render() {
    return( 
    <div>
      <button onClick={this.clickhandler}> // => CALLBACK
        Click me
      </button>
    </div>)
  }
}
```

看起来使用箭头函数能解决我们的问题，但是，我们失去了什么？ 我们总是需要交易一些东西以获得正确的收益，让我们深入看看这两个类。

```
// Assume FooRegularFunction defines clickhandler as regular function and FooArrowFunction defines clickhandler
 
console.dir(FooRegularFunction)
console.dir(FooArrowFunction)
```

我们看到，clickhandler被分配给 FooRegularFunction 的原型属性（假设 App 类和 FooRegularFunction 类是相同的的）。

![img](https://pic3.zhimg.com/80/v2-28a7f65963dc2b5f48c0b339b8d2e5ae_hd.jpg)FooRegularFunction

但对于FooArrowFunction，我们没有看到 clickhandler。 它未分配给prototype属性。 那它在哪里呢？

![img](https://pic2.zhimg.com/80/v2-f20a739f4d3ecdd71ca896fd1f7032d9_hd.jpg)FooArrowFunction

为了理解这一点，我们将了解另一个微妙的主题。

```
class Foo {
  constructor(name) {
    this.name = name
  }
  function bar() {console.log(this.name)}
  foobar = () => {console.log(this.name)}
}
var f = new Foo("jack")
f.bar() // jack
f.foobar() // jack
// <see images below>
console.dir(Foo) 
console.dir(f)
```

在上面的例子中，我们有两个类方法，一个定义为常规函数，另一个定义为箭头函数。 让我们看一下类和实例的属性。

![img](https://pic4.zhimg.com/80/v2-dd5fc8f82dab8b381b92f4935f5db9bf_hd.jpg)

在上图中，第一部分是 `Foo` 类属性，第二部分是 `Foo` 实例属性。 和我们预想的一致，作为常规函数的 `bar()` 附加到类定义的原型，并在创建实例时被复制到 **proto** 属性。 但是，`foobar()`并没有出现在 `Foo` 类属性中，而是被定义为实例上的独立属性。

这正是箭头函数作为类方法工作正常的原因。 但是，该方法在类属性中并没有出现。 但是，为什么它隐藏了呢？

在 ES Next 上，你会明白，这是这是 JavaScript 标准委员会 [TC39](http://link.zhihu.com/?target=https%3A//tc39.github.io/beta/) 提出的一个[实验性功能（第3阶段）](http://link.zhihu.com/?target=https%3A//github.com/tc39/proposal-class-fields)。这正是为了带来面向对象编程的超级特征(super-star feature)的巨大优势而提出的，即“封装”。

我会在将来再展开这个话题

最后，我想通过一句陈述来结束这篇文章

> **“用箭头函数替换定义为常规函数的类方法能够有效地使类方法成为私有的（不完全正确）。 这种方法的利弊在具体情况下是需要主观衡量的。“**