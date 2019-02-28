---
title: 谈谈 Node.js 中的事件循环
date: 2019-02-28 16:50:44
tags: 
  - node.js
---

## 前言
最近面试中提到 js 的事件循环，在浏览器内的事件循环已经说过很多遍了，无非是主线程任务队列宏任务微任务等，但是在 Node.js 中的事件循环机制一直没有系统梳理过，只有个大概印象，就借这个机会整理一下。

## 循环阶段
Node.js 中的事件循环由 `libuv` 实现，其每一轮循环按照顺序氛围6个阶段：
1. timers：定时器阶段，执行 `setTimeout()` 和 `setInterval()` 的回调
2. I/O pending callbacks: 执行从上一轮 poll 遗留的，已完成I/O操作的回调函数
3. idle prepare: 系统内部使用，可忽略
4. poll: 等待还没完成的I/O事件，会因timers和超时时间等结束等待
5. check: 执行 `setImmediate()` 回调函数
6. close callbacks: 处理一些准备关闭的回调函数

## 执行机制 
### 任务队列
不同于浏览器中的 microtask 和 macrotask，node 中具有的两个队列分别是 microtask queue 和 nexttick queue
在循环阶段中的几个队列分别是 Timers Queue、I/O Queue、Check Queue、Close Queue

###循环之前
在进入第一次循环之前，会先进行如下操作：

- 同步任务
- 发出异步请求
- 规划定时器生效的时间
- 执行process.nextTick()

### 开始循环

按照我们的循环的6个阶段依次执行，每次拿出当前阶段中的全部任务执行，清空NextTick Queue，清空Microtask Queue。再执行下一阶段，全部6个阶段执行完毕后，进入下轮循环。即：

- 清空当前循环内的Timers Queue，清空NextTick Queue，清空Microtask Queue。
- 清空当前循环内的I/O Queue，清空NextTick Queue，清空Microtask Queue。
- 清空当前循环内的Check Queue，清空NextTick Queue，清空Microtask Queue。
- 清空当前循环内的Close Queue，清空NextTick Queue，清空Microtask Queue。
- 进入下轮循环。

可以看出，`nextTick` 优先级比 `promise` 等 `microtask` 高。`setTimeout`和`setInterval`优先级比`setImmediate`高。

## 注意

如果在 timers 阶段执行时创建了 `setImmediate` 则会在此轮循环的 `check` 阶段执行，如果在 timers 阶段创建了 `setTimeout` ，由于timers 已取出完毕，则会进入下轮循环，check 阶段创建 timers 任务同理。
`setTimeout` 优先级比setImmediate高，但是由于 `setTimeout(fn,0)` 的真正延迟不可能完全为0秒，可能出现先创建的 `setTimeout(fn,0)` 而比 `setImmediate` 的回调后执行的情况。

## 测试代码

```javascript
function sleep(time) {
  let startTime = new Date()
  while (new Date() - startTime < time) {}
  console.log('1s over')
}
setTimeout(() => {
  console.log('setTimeout - 1')
  setTimeout(() => {
      console.log('setTimeout - 1 - 1')
      sleep(1000)
  })
  new Promise(resolve => resolve()).then(() => {
      console.log('setTimeout - 1 - then')
      new Promise(resolve => resolve()).then(() => {
          console.log('setTimeout - 1 - then - then')
      })
  })
  sleep(1000)
})

setTimeout(() => {
  console.log('setTimeout - 2')
  setTimeout(() => {
      console.log('setTimeout - 2 - 1')
      sleep(1000)
  })
  new Promise(resolve => resolve()).then(() => {
      console.log('setTimeout - 2 - then')
      new Promise(resolve => resolve()).then(() => {
          console.log('setTimeout - 2 - then - then')
      })
  })
  sleep(1000)
})
```
浏览器输出:
```
setTimeout - 1 //1为单个task
1s over
setTimeout - 1 - then
setTimeout - 1 - then - then 
setTimeout - 2 //2为单个task
1s over
setTimeout - 2 - then
setTimeout - 2 - then - then
setTimeout - 1 - 1
1s over
setTimeout - 2 - 1
1s over
```
node 输出:
```
setTimeout - 1 
1s over
setTimeout - 2 //1、2为单阶段task
1s over
setTimeout - 1 - then
setTimeout - 2 - then
setTimeout - 1 - then - then
setTimeout - 2 - then - then
setTimeout - 1 - 1
1s over
setTimeout - 2 - 1
1s over
```