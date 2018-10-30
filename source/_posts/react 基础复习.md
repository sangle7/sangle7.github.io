---
title: react 基础复习
date: 2017-08-24 15:50:44
tags: 
  - react
  - 面试
---


## 概述

### react解决了什么问题

a. React实现了Virtual DOM

在一定程度上提升了性能，尤其是在进行小量数据更新时。因为DOM操作是很耗性能的，而Virtual DOM是在内存中进行操作的，当数据发生变化时，通过diff算法比较两棵树之间的变化，再进行必要的DOM更新，省去了不必要的高消耗的DOM操作。当然，这种性能优化主要体现在有小量数据更新的情况下。因为React的基本思维模式是每次有变动就重新渲染整个应用，简单想来就是直接重置innerHTML，比如说在一个大型列表所有数据都变动的情况下，重置innerHTML还比较合理，但若是只有一行数据变了，它也需要重置整个innerHTML，就会造成大量的浪费。而Virtual DOM虽然进行了JS层面的计算，但是比起DOM操作来说，简直不要太便宜。

b. React的一个核心思想是声明式编程。

命令式编程是解决做什么的问题，就像是下命令一样，关注于怎么做，而声明式编程关注于得到什么结果，在React中，我们只需要关注“目前的状态是什么”，而不是“我需要做什么让页面变成目前的状态”。React就是不断声明，然后在特定的参数下渲染UI界面。这种编程方式可以让我们的代码更容易被理解，从而易于维护。

<!-- more -->

c. 组件化

React天生组件化，我们可以将一个大的应用分割成很多小组件，这样有好几个优势。首先组件化的代码像一棵树一样清楚干净，比起传统的面条式代码可读性更高；其次前端人员在开发过程中可以并行开发组件而不影响，大大提高了开发效率；最重要的是，组件化使得复用性大大提高，团队可以沉淀一些公共组件或工具库。

d. 单向数据流

在React中数据流是单向的，由父节点流向子节点，如果父节点的props发生了变化，那么React会递归遍历整个组件树，重新渲染所有使用该属性的子组件。这种单向的数据流一方面比较清晰不容易混乱，另一方面是比较好维护，出了问题也比较好定位。


## setstate/diff

### setState后发生了什么

- 在代码中调用setState函数之后，React 会将传入的参数对象与组件当前的状态合并，然后触发所谓的调和过程（Reconciliation）。经过调和过程，React 会以相对高效的方式根据新的状态构建 React 元素树并且着手重新渲染整个UI界面。在 React 得到元素树之后，React 会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算算法中，React 能够相对精确地知道哪些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

### diff算法的假设

使得Diff算法复杂度从O(n^3)降低到O(n)

1. 两个相同组件产生类似的DOM结构，不同的组件产生不同的DOM结构；
2. 对于同一层次的一组子节点，它们可以通过唯一的id进行区分。

### 传入 setState 函数的第二个参数的作用是什么？

该函数会在setState函数调用完成并且组件开始重渲染的时候被调用，我们可以用该函数来监听渲染是否完成：

this.setState(  { username: 'tylermcginnis33' },  () => console.log('setState has finished and the component has re-rendered.'))

### 下述代码有错吗？

this.setState((prevState, props) => {  return {

```
streak: prevState.streak + props.count
```

  }})这段代码没啥问题，不过只是不太常用罢了，详细可以参考React中setState同步更新策略

## 生命周期

### 组件的生命周期

组件生命周期有三种阶段：初始化阶段（Mounting）、更新阶段（Updating）、析构阶段（Unmouting）。

初始化阶段：

constructor()：初始化state、绑定事件componentWillMount()：在render()之前执行，除了同构，跟constructor没啥差别render()：用于渲染DOM。如果有操作DOM或和浏览器打交道的操作，最好在下一个步骤执行。componentDidMount()：在render()之后立即执行，可以在这个函数中对DOM就进行操作，可以加载服务器数据，可以使用setState()方法触发重新渲染组件更新阶段：

componentWillReceiveProps(nextProps)：在已挂载的组件接收到新props时触发，传进来的props没有变化也可能触发该函数，若需要实现props变化才执行操作的话需要自己手动判断componentShouldUpdate(nextProps，nextState)：默认返回true，我们可以手动判断需不需要触发render，若返回false，就不触发下一步骤componentWillUpdate()：componentShouldUpdate返回true时触发，在render之前，可以在里面进行操作DOMrender()：重渲染componentDidUpdate()：render之后立即触发组件卸载阶段：

componentWillUnmount()：在组件销毁之前触发，可以处理一些清理操作，如无效的timers等componentDidMount()：卸载后立即触发

### shouldComponentUpdate 的作用是啥以及为何它这么重要？

shouldComponentUpdate 允许我们手动地判断是否要进行组件更新，根据组件的应用场景设置函数的合理返回值能够帮我们避免不必要的更新。

### 在哪些生命周期中可以修改组件的state

componentDidMount和componentDidUpdateconstructor、componentWillMount中setState会发生错误：setState只能在mounted或mounting组件中执行componentWillUpdate中setState会导致死循环

### 在生命周期中的哪一步你应该发起 AJAX 请求？

我们应当将AJAX 请求放到 componentDidMount 函数中执行，主要原因有下：

React 下一代调和算法 Fiber 会通过开始或停止渲染的方式优化应用性能，其会影响到 componentWillMount 的触发次数。对于 componentWillMount 这个生命周期函数的调用次数会变得不确定，React 可能会多次频繁调用 componentWillMount。如果我们将 AJAX 请求放到 componentWillMount 函数中，那么显而易见其会被触发多次，自然也就不是好的选择。

如果我们将 AJAX 请求放置在生命周期的其他函数中，我们并不能保证请求仅在组件挂载完毕后才会要求响应。如果我们的数据请求在组件挂载之前就完成，并且调用了setState函数将数据添加到组件状态中，对于未挂载的组件则会报错。而在 componentDidMount 函数中进行 AJAX 请求则能有效避免这个问题。

### 组件的render函数何时被调用

组件state发生改变时会调用render函数，比如通过setState函数改变组件自身的state值继承的props属性发生改变时也会调用render函数，即使改变的前后值一样React生命周期中有个componentShouldUpdate函数，默认返回true，即允许render被调用，我们也可以重写这个函数，判断是否应该调用render函数

### 调用render时DOM就一定会被更新吗

不一定更新。

React组件中存在两类DOM，render函数被调用后， React会根据props或者state重新创建一棵virtual DOM树，虽然每一次调用都重新创建，但因为创建是发生在内存中，所以很快不影响性能。而 virtual dom的更新并不意味着真实DOM的更新，React采用diff算法将virtual DOM和真实DOM进行比较，找出需要更新的最小的部分，这时Real DOM才可能发生修改。

所以每次state的更改都会使得render函数被调用，但是页面DOM不一定发生修改。

## 组件

### React 中 Element 与 Component 的区别是？

简单而言，React Element 是描述屏幕上所见内容的数据结构，是对于 UI 的对象表述。典型的 React Element 就是利用 JSX 构建的声明式代码片然后被转化为createElement的调用组合。而 React Component 则是可以接收参数输入并且返回某个 React Element 的函数或者类。更多介绍可以参考React Elements vs React Components。

### 在什么情况下你会优先选择使用 Class Component 而不是 Functional Component？

在组件需要包含内部状态或者使用到生命周期函数的时候使用 Class Component ，否则使用函数式组件。

### React 中 refs 的作用是什么？

注意，根据React最新文档，下面这种用法已经被弃用了，统一改为回调函数模式this.refs.textInputRefs 是 React 提供给我们的安全访问 DOM 元素或者某个组件实例的句柄。我们可以为元素添加ref属性然后在回调函数中接受该元素在 DOM 树中的句柄，该值会作为回调函数的第一个参数返回：

### React 中 keys 的作用是什么？

Keys 是 React 用于追踪哪些列表中元素被修改、被添加或者被移除的辅助标识。

在开发过程中，我们需要保证某个元素的 key 在其同级元素中具有唯一性。在 React Diff 算法中 React 会借助元素的 Key 值来判断该元素是新近创建的还是被移动而来的元素，从而减少不必要的元素重渲染。此外，React 还需要借助 Key 值来判断元素与本地状态的关联关系，因此我们绝不可忽视转换函数中 Key 的重要性。

### createElement 与 cloneElement 的区别是什么？

createElement 函数是 JSX 编译之后使用的创建 React Element 的函数，而 cloneElement 则是用于复制某个元素并传入新的 Props。

### 如何对组件进行优化

使用上线构建（Production Build）：会移除脚本中不必要的报错和警告，减少文件体积避免重绘：重写shouldComponentUpdate函数，手动控制是否应该调用render函数进行重绘尽可能使用Immutable Data：尽可能不修改数据，而是重新赋值数据。这样在检测数据对象是否发生修改方面会非常快，因为只需要检测对象引用即可，不需要挨个检测对象属性的更改在渲染组件时尽可能添加key，这样virtual DOM在对比的时候就更容易知道哪里是修改元素，哪里是新插入的元素

### 受控组件（controlled component）和不受控组件（uncontrolled component）有什么区别

受控组件：不受控组件：表单数据不受setState控制，而是由DOM本身处理，与传统HTML表单输入相似，input输入值即显示最新值（使用ref从DOM获取表单值）

### Component与Element和Instance的区别

Element其实是一个纯粹的Object对象，用于描述在屏幕上看到的DOM节点，这个对象包括type、props、key和ref属性，但不包括DOM方法（React.createElement()）Component是组件级别的类：接收参数并返回React元素的函数或类Instance：当使用ReactDOM.render()将一个组件渲染到一个具体的DOM元素中，返回的值就为一个实例

### 什么是React的refs，为什么它们很重要

我们用render方法得到了组价的实例，然后就可以对它进行相关操作，但是在组件内，JSX是不会返回一个组件的实例，它只是一个ReactElement，只是告诉React被挂载的组件应该是长什么样。

refs是组件的一个很特殊的prop，可以附加到任何一个组件上，refs就是reference，组件被调用时会新建一个该组件的实例，refs就会指向这个实例

我们把refs放到原生的DOM组件input中，就可以通过refs得到DOM结点；如果把refs放到React组件中，就可以获得组件的实例，可以调用该组件的实例方法

### 无状态组件

无状态组件不支持 "ref"

有一点遗憾的是无状态组件不支持 "ref"。原理很简单，因为在 React 调用到无状态组件的方法之前，是没有一个实例化的过程的，因此也就没有所谓的 "ref"。

ref 和 findDOMNode 这个组合，实际上是打破了父子组件之间仅仅通过 props 来传递状态的约定，是危险且肮脏，需要避免。

### 高阶组件

- 当写着写着无状态组件的时候，有一天忽然发现需要状态处理了，那么无需彻底返工:)
- 往往我们需要状态的时候，这个需求是可以重用的。原理：柯里化（curry）

## 生产

### 如何告诉 React 它应该编译生产环境版本？

通常情况下我们会使用 Webpack 的 DefinePlugin 方法来将 NODE_ENV 变量值设置为 production。编译版本中 React 会忽略 propType 验证以及其他的告警信息，同时还会降低代码库的大小，React 使用了 Uglify 插件来移除生产环境下不必要的注释等信息。

## 事件

### 概述下 React 中的事件处理逻辑

为了解决跨浏览器兼容性问题，React 会将浏览器原生事件（Browser Native Event）封装为合成事件（SyntheticEvent）传入设置的事件处理器中。这里的合成事件提供了与原生事件相同的接口，不过它们屏蔽了底层浏览器的细节差异，保证了行为的一致性。另外有意思的是，React 并没有直接将事件附着到子元素上，而是以单一事件监听器的方式将所有的事件发送到顶层进行处理。这样 React 在更新 DOM 的时候就不需要考虑如何去处理附着在 DOM 上的事件监听器，最终达到优化性能的目的。

### 描述事件在React中的处理方式

React在virtual DOM的基础上实现了一个SyntheticEvent（合成事件）层，所有事件都绑定到最外层上。

在React底层，主要对合成事件做了两件事：事件委托和自动绑定。

事件委托：React的事件代理机制不会把事件处理函数直接绑定到真实的结点上，而是把所有事件绑定到结构的最外层，使用统一的事件监听器，这个事件监听器上维持了一个映射来保存所有组件内部的事件监听和处理函数。当事件发生时，首先被这个统一的事件监听器处理，然后在映射里找到真正的事件处理函数并调用。

自动绑定：在React组件中，每个方法的上下文都会指向该组件的实例，即自动绑定this为当前组件。在使用ES6 classes和纯函数时，这种自动绑定就不存在了，需要我们手动绑定this：bind方法、双冒号语法、构造器内声明、箭头函数。

## redux

1. 单一数据源，Single Source of Truth（也即题干中提到的 「单一的 State 状态树」）
2. 所有数据都是只读的，要想修改数据，必须 dispatch 一个 action 来描述什么发生了改变
3. 当处理 action 时，必须生成一个新的 state，不得直接修改原始对象

## react-router

### 使用react-router的项目即使切换了页面，URL发生了改变，但是页面也不会刷新，这是如何实现的？

答：react-router只会用基础的部分，一直只知道用它来管理路由，但是确实不知道实现原理，直接和面试官说确实没有深入了解过，然后就算了。    其实页面没刷新，但页面内容却改变了，这不就是ajax吗？ 

### react-router 原理

createBrowserHistory: 利用HTML5里面的history执行URL前进 => createBrowserHistory: pushState、replaceState检测URL回退 => createBrowserHistory: popstate为了维护state的状态，将其存储在sessionStorage里面:![img](http://onvaoy58z.bkt.clouddn.com/7cdsxs0sxza.png)