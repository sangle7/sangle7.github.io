---
title: React中的 Render Props
date: 2017-11-16 15:50:44
tags: 
  - react
---

最近 render props 在外网挺火，在逛 medium 的时候看到好几篇文章都是关于它的，由于笔者目前在做毕业设计，业务逻辑上不难实现，所以在这个过程中更多的考虑去尝试一些新的东西。在详细阅读了几篇文章后笔者也在自己的项目中用上了render props，这里就说说在这个过程中的一些思考。

## 解决了什么问题？

最重要的一点，render props 其实和高阶组件HOC一样，是为了给纯函数组件加上state，响应react的生命周期。

为什么会有这样的需求呢？我们要从HOC说起

<!-- more -->

## 先谈谈HOC

我们写的纯函数组件只负责处理展示，很多时候会发现，由于业务需求，组件需要被“增强”，例如响应浏览器事件等。如果只有一两个组件我们大可以全部重写为class形式，但如果有许多组件需要进行相似或相同的处理（例如都响应浏览器窗口改变这个事件）时，考虑到代码的复用性，很容易想到用函数处理，HOC也正是为了解决这样的问题而出现的。

### 基本原理

HOC的基本原理可以写成这样：

```javascript
const HOCFactory = (Component) => {
  return class HOC extends React.Component {
    render(){
      return <Component {...this.props} />
    }
  }
}
```

很明显HOC最大的特点就是：接受一个组件作为参数，返回一个新的组件。

### 简单例子

这里是一个响应鼠标事件的HOC例子：

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

const withMouse = (Component) => {
  return class extends React.Component {
    state = { x: 0, y: 0 }

    handleMouseMove = (event) => {
      this.setState({
        x: event.clientX,
        y: event.clientY
      })
    }

    render() {
      return (
        <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>
          <Component {...this.props} mouse={this.state}/>
        </div>
      )
    }
  }
}

const App = (props) => {
  const { x, y } = props.mouse
  return (
    <div style={{ height: '100%' }}>
      <h1>The mouse position is ({x}, {y})</h1>
    </div>
  ) 
}

const AppWithMouse = withMouse(App)

ReactDOM.render(<AppWithMouse/>, document.getElementById('root'))
```

### 优劣分析

笔者曾经使用了比较长时间的HOC，让我坚持用下去大概有以下这些原因，

支持ES6，光这一项就战胜了mixins
复用性强，HOC是纯函数且返回值仍为组件，在使用时可以多层嵌套，在不同情境下使用特定的HOC组合也方便调试。
同样由于HOC是纯函数，支持传入多个参数，增强了其适用范围。
当然HOC也存在一些问题（不然我就不会写这篇文章了...）

当有多个HOC一同使用时，无法直接判断子组件的props是哪个HOC负责传递的。
重复命名的问题：若父子组件有同样名称的props，或使用的多个HOC中存在相同名称的props，则存在覆盖问题，而且react并不会报错。当然可以通过规范命名空间的方式避免。
在react开发者工具中观察HOC返回的结构，可以发现HOC产生了许多无用的组件，加深了组件层级。
同时，HOC使用了静态构建，即当AppWithMouse被创建时，调用了一次withMouse中的静态构建。而在render中调用构建方法才是react所倡导的动态构建。与此同时，在render中构建可以更好的利用react的生命周期。
render prop 的出现解决了以上问题

## Render Props

> A render prop is a function prop that a component uses to know what to render.

Render Props 的核心思想是，通过一个函数将class组件的state作为props传递给纯函数组件

### 简单例子

这里是一个简单的使用render prop的例子

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import PropTypes from 'prop-types'

class Mouse extends React.Component {
  static propTypes = {
    render: PropTypes.func.isRequired
  }

  state = { x: 0, y: 0 }

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    })
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    )
  }
}

const App = () => (
  <div style={{ height: '100%' }}>
    <Mouse render={({ x, y }) => (
      <h1>The mouse position is ({x}, {y})</h1>
    )}/>
  </div>
)

ReactDOM.render(<App/>, document.getElementById('root'))
```

### 核心分析

从demo中很容易看到，新建的Mouse组件的render方法中返回了`{this.props.render(this.state)}`这个函数，将其state作为参数传入其的props.render方法中，调用时直接取组件所需要的state即可。

这样做的优势是

支持ES6，和HOC一样
不用担心prop的命名问题，在render函数中只取需要的state
相较于HOC，不会产生无用的空组件加深层级
最重要的是，这里的构建模型是动态的，所有改变都在render中触发，能更好的利用react的生命周期。

[MICHAEL JACKSON‏](https://zhuanlan.zhihu.com/p/31267131/%3C/b%3E//twitter.com/mjackson) 在twitter上发起挑战，称所有HOC都能改写为Render props，如果你发现有例外，欢迎[提出质疑](https://link.zhihu.com/?target=https%3A//twitter.com/mjackson/status/885910701520207872) 😉

![](http://onvaoy58z.bkt.clouddn.com/v2-c16db5d725247c79746c3fb1b70b4054_hd.jpg)


## 选择

对于二者之间的选择问题，笔者持中立态度，一方面render props 在大多数情况下比HOC更直观也更利于调试，另一方面HOC可传入多个参数的特性让我能少写不少代码，在目前的这个项目中我二者都有使用，个人根据不同的场景进行选择更有利于开发效率的提高。


---

拓展阅读：

[https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)

[https://reactrocket.com/post/turn-your-hocs-into-render-prop-components/](https://reactrocket.com/post/turn-your-hocs-into-render-prop-components/)

[https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9](https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9)




