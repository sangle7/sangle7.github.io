---
title: 从 React 的一个 bug 谈起……
date: 2018-01-11 17:50:44
tags: 
  - react
---


前几天在项目中遇到一个 React（v15.6.2）的 bug，查了一下 issue 确实有人提过了，**并且已经在v16中被修复了**， 本着对技术的好奇我深入研究了一下源码，这篇文章就从这里说起……

## 问题描述

简单来说是这样的 (或者你可以直接看这个[demo](https://stackblitz.com/edit/react-onchange-bug)) ：

```javascript
const App = props => (
  <form onChange={e => console.log(e.target.value)}>
    <label>
      <input type={'radio'} name={'myGroup'} value={1}/>
      Value 1
    </label>
    <label>
      <input type={'radio'} name={'myGroup'} value={2}/>
      Value 2
    </label>
  </form>
)

ReactDOM.render(<App />,document.getElementById('root'))
```

<!-- more -->

在这段代码中，我们的预期是每次点击 radio 切换选择值的时候，都会触发 form 上的 onchange 事件，输出 e.target.value。然而情况却是 onchange 事件只会在 每个 radio 被第一次选中时触发。也就是说，如果我们选择了1，2，1，onchange 事件只会被触发两次。

## 问题追溯

由于是事件相关的bug，我们把问题定位在 react-dom 库中，先来看看代码是怎么写的：）

###ChangeEventPlugin.js

首先由于我们监听的是 onchange 事件，很容易注意到 `ChangeEventPlugin.js` 这个文件，在 react-dom 中，这个模块负责处理 changeEvent

这里额外说两句，react-dom 中的事件都是通过插件的形式处理的，在`ReactDefaultInjection.js` 中，通过这样一段代码在全局默认注入了这些事件处理插件：

```javascript
  /**
   * Some important event plugins included by default (without having to require
   * them).
   */
  ReactInjection.EventPluginHub.injectEventPluginsByName({
    SimpleEventPlugin: SimpleEventPlugin,
    EnterLeaveEventPlugin: EnterLeaveEventPlugin,
    ChangeEventPlugin: ChangeEventPlugin,
    SelectEventPlugin: SelectEventPlugin,
    BeforeInputEventPlugin: BeforeInputEventPlugin
  });
```

继续查看 `ChangeEventPlugin.js` ，去除无关代码，我们先来梳理一下整个change event的逻辑（代码已简化）：

```javascript
function getInstIfValueChanged(targetInst, nativeEvent) {
  /* 判断实例的value是否变化，如果变化则返回该实例 */
  var updated = inputValueTracking.updateValueIfChanged(targetInst);
  var simulated = nativeEvent.simulated === true && ChangeEventPlugin._allowSimulatedPassThrough;
  if (updated || simulated) {
    return targetInst;
  }
}

function shouldUseClickEvent(elem) {
  /* 判断是否应该使用click事件处理 */
  var nodeName = elem.nodeName;
  return nodeName && nodeName.toLowerCase() === 'input' && (elem.type === 'checkbox' || elem.type === 'radio');
}

function getTargetInstForClickEvent(topLevelType, targetInst, nativeEvent) {
   /* 获取click事件的目标实例 */
  if (topLevelType === 'topClick') {
    return getInstIfValueChanged(targetInst, nativeEvent);
  }
}


var ChangeEventPlugin = {
  extractEvents: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    var targetNode = targetInst ? ReactDOMComponentTree.getNodeFromInstance(targetInst) : window;

    var getTargetInstFunc, handleEventFunc;
    if (shouldUseClickEvent(targetNode)) {
      getTargetInstFunc = getTargetInstForClickEvent;
    }

    if (getTargetInstFunc) {
      var inst = getTargetInstFunc(topLevelType, targetInst, nativeEvent);
    }
  }
};

module.exports = ChangeEventPlugin;
```

第一个函数对实例的取值是否变化进行了判断，我们先把问题定位到这里

```javascript
function getInstIfValueChanged(targetInst, nativeEvent) {
  var updated = inputValueTracking.updateValueIfChanged(targetInst);
  var simulated = nativeEvent.simulated === true && ChangeEventPlugin._allowSimulatedPassThrough;

  if (updated || simulated) {
    return targetInst;
  }
}
```

通过打断点或是打 log 的方式，我们注意到在 updated = true 时会触发 onchange 事件，而 updated = false 则不会，那么为什么在每个 radio 在第 n(n>1) 次被选中时的 updated 会为 false 呢？我们接着来看 `inputValueTracking.updateValueIfChanged` 这个函数

### inputValueTracking.js

顾名思义，该文件用于跟踪 input 的 value，同样先简单的看一下：

```javascript
var inputValueTracking = {
  track: function (inst) {
      /* 跟踪value */
  },

  updateValueIfChanged: function (inst) {
   /* 判断 value 是否发生变化*/
    var tracker = getTracker(inst);
    var lastValue = tracker.getValue();
    var nextValue = getValueFromNode(ReactDOMComponentTree.getNodeFromInstance(inst));
    if (nextValue !== lastValue) {
      tracker.setValue(nextValue);
      return true;
    }

    return false;
  },
  
  stopTracking: function (inst) {
      /* 停止跟踪 */
  }
};
module.exports = inputValueTracking;
```

emmm，原来 lastValue 在点击其他 radio 后根本没变！难怪我们的 onchange 事件只能被触发一次，似乎终于有点接近问题的真相了……

在这里打个日志就会发现，每次我们点击都只会触发一次`updateValueIfChanged` 函数，试想一个正常的逻辑，应该是在我们每次点击 radio 时，会对每个 radio 都触发该函数，这样才能将本来处于选中状态的 radio ( lastValue = true ) 变为非选中状态  ( thisValue = false )

## 问题解决

接着看下来，我们发现其实react 对于这个问题是做了处理的，在 `ReactDOMComponent.js`  中有这样一段：

```javascript
updateComponent:(){
    switch (this._tag) {
          case 'input':
            ReactDOMInput.updateWrapper(this);
            // We also check that we haven't missed a value update, such as a Radio group shifting the checked value to another named radio input.
            inputValueTracking.updateValueIfChanged(this);
            break;
    }
}
```

随便打个断点就会发现这段代码并没有执行，原来是触发`updateComoponent`的逻辑是这样的：

```javascript
/* Receives a next element and updates the component. */
  receiveComponent: function (nextElement, transaction, context) {
    var prevElement = this._currentElement;
    this._currentElement = nextElement;
    this.updateComponent(transaction, prevElement, nextElement, context);
},
```

 嗯……我们想要的是对于同一个 name 下的 radio ，在每次改变取值的时候都会触发所有 radio 的 `inputValueTracking.updateValueIfChanged` 方法。其中的首要条件是这些 radio 的 name 属性值相同，根据这些条件我们定位到 `ReactDOMInput.js` 中的 `_handleChange` 函数：

```javascript
function _handleChange(event) {
  var props = this._currentElement.props;

  var returnValue = LinkedValueUtils.executeOnChange(props, event);

  var name = props.name;
  if (props.type === 'radio' && name != null) {
    var rootNode = ReactDOMComponentTree.getNodeFromInstance(this);
    var queryRoot = rootNode;

    while (queryRoot.parentNode) {
      queryRoot = queryRoot.parentNode;
    }

    var group = queryRoot.querySelectorAll('input[name=' + JSON.stringify('' + name) + '][type="radio"]');

    for (var i = 0; i < group.length; i++) {
      var otherNode = group[i];
      if (otherNode === rootNode || otherNode.form !== rootNode.form) {
        continue;
      }
      
      var otherInstance = ReactDOMComponentTree.getInstanceFromNode(otherNode);
      !otherInstance ? process.env.NODE_ENV !== 'production' ? invariant(false, 'ReactDOMInput: Mixing React and non-React radio inputs with the same `name` is not supported.') : _prodInvariant('90') : void 0;
        /* add */
      ReactUpdates.asap(forceUpdateIfMounted, otherInstance);
    }
  }
```

这段代码中遍历了同个 name 下的所有 radio，并将除了被触发动作的 radio 外的其他 radio 实例储存在 `otherInstance` 变量内，我们就在这里触发`inputValueTracking.updateValueIfChanged` 方法

在 `/* add */ `处加上一行`inputValueTracking.updateValueIfChanged(otherInstance);`，并在文件内引入相应变量，`npm build` ，问题解决！

## 单元测试

为了确认问题是否真的解决了，我们在 react 源码上进行修改后，至少要进行单元测试验证一下，具体可以查看 [Sending a Pull Request](https://reactjs.org/docs/how-to-contribute.html#sending-a-pull-request)，按照说明安装`yarn`，执行`yarn test`：

![](https://s1.ax1x.com/2018/02/11/9G4HVe.png)

嗯，单元测试通过了，接下来要添加一个新的测试用例以证明我们的修改是必要的：

（不要问我为什么没写完因为我要去赶车回家了）



