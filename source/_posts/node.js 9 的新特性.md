---
title: Node.js 9 的新特性
date: 2018-02-07 18:58:44
tags: 
    - Node.js
---

这篇文章原定是去年11月写的，现在三个月过去了…v9.5都出了，这里就一并总结一下，看看Node在最新的版本中有什么变化。

Node.js 每年进行两次大的发布，2017年10月，Node.js发布了 9.0 版本，与此同时， Node.js 8.9.0 成为了最新的 LTS 版本。这意味着对 8.9.0 的支持将会维持到2019年底，此后的一个LTS版本将会是 Node.js 10

<!-- more -->

![](http://onvaoy58z.bkt.clouddn.com/v2-862ad2d8c02ca650c45203c0a47eb2a2_hd.jpg)

查看 [Node.js 发布历史](https://github.com/nodejs/Release)



##  Node.js 9 有什么新特性？

### http/2

node 8.4 首次支持了http/2

```javascript
const http2 = require('http2');
const server = http2.createServer();
server.on('stream', (stream, requestHeaders) => {
  stream.respond({ ':status': 200, 'content-type': 'text/plain' });
  stream.write('hello ');
  stream.end('world');
});
server.listen(8000);
```

但由于仍处于实验性阶段，运行时需要加上`—expose-http2` 参数

```bash
$ node --expose-http2 h2server.js
```

而在node 9中去除了这一参数，可以直接使用

```bash
$ node h2server.js
```

当然由于现在主流浏览器只有在HTTPS时才启用HTTP2，我们要启动一个https的服务器，可以参考[如何在本地搭建https服务器](https://medium.freecodecamp.org/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec)

其他关于http/2的变化还包括：

1. 新增对 ALTSVC([HTTP Alternative Services](https://tools.ietf.org/html/rfc7838)) 的支持
2. 新增 `maxSessionMemory`  ，限制单个http2线程允许使用的内存上限，一旦超过这个值，http2请求将被拒绝
3. 收集并报告有关 `Http2Session`  和 `Http2Stream` 实例的基本计时信息。
4. 改进了`Http2Stream`和`Http2Session`的关闭方式，`Http2Stream.prototype.rstStream()` 方法被移动至 `Http2Stream.prototype.close()` 中
5. 在`Http2Session`上引入了新的属性来确定会话是否安全

关于http/2:

[Node.js HTTP/2 documentation](https://nodejs.org/api/http2.html)

[Introduction to HTTP/2](https://developers.google.com/web/fundamentals/performance/http2/)

### util

1. 新增方法`util.isDeepStrictEqual(value1, value2)`，可进行两个值的深度比较，返回一个布尔值。此前我们常用`assert.deepStrictEqual()`进行比较，后者在两个值不相等时会抛出异常。

   ```javascript
   const { isDeepStrictEqual } = require('util');
   const isEqual = isDeepStrictEqual({	a: '1' }, { a: 1 });
   console.log(isEqual);	//false

   const { deepStrictEqual } = require('assert')
   const isEqual = deepStrictEqual({	a: '1' }, { a: 1 });
   // throw new errors.AssertionError
   ```

2. 新增方法`util.callbackify`，可以将 Promise 转化为callback形式的函数，适用于解决一些兼容性问题的场景：

   ```javascript
   const { callbackify } = require('util')
   async function promiseDemo () {
     await Promise.resolve()
   }
   callbackify(promiseDemo)(function (err) {
     if (err) {
       return console.error(err)
     }
     console.log('finished without an error')
   })
   ```

3. 允许在 `debuglog()` 中使用通配符，通过 `NODE_DEBUG=foo*` 环境运行

  ```javascript
  const util = require('util');
  const debuglog = util.debuglog('foo-bar');

  debuglog('hi there, it\'s foo-bar [%d]', 2333);
  ```

### HTTP/1

1. 当传入的请求无法成功解析时，http模块现在将返回400状态码。在过去，Node.js只会挂断socket，导致其他服务器（如nginx）误以为Node.js服务器关闭。
2. 在此前的版本中，一旦套接字被分配给请求，`request.setTimeout()`就会调用`socket.setTimeout()`。 这使得即使底层的套接字永不连接，也会在请求上发出超时事件。在 Node.js 9 中，`socket.setTimeout()`仅在底层套接字成功连接时被调用。
3. 新增103状态码：[103 Early Hints](https://datatracker.ietf.org/meeting/97/materials/slides-97-httpbis-sessb-early-hints/) 该状态码允许服务器在主报头之前先发送部分报头，以达到预加载文件的目的。

### 更严谨的错误码

Node.js核心代码库正在慢慢迁移到一个新的错误系统，Node.js 9 中采用了更严谨的错误码

在Node.js 9 之前，你可能会这样处理错误:

```javascript
if (err.message === 'Console expects a writable stream instance') {
  //do something with the error
}
```

现在则应该用这种方式进行处理:

```javascript
if (err.code === 'ERR_CONSOLE_WRITABLE_STREAM') {
  //do something with the error
}
```

这可能会导致升级中的各种问题，为了使用户平滑升级，Node.js官方给出了它们采用的错误码，可以在[这里](https://nodejs.org/api/errors.html#errors_node_js_error_codes)查看


### 其他变化

1. assert 模块的方法现在可以抛出任何类型的错误（RangeError，SyntaxError等）。在之前版本的Node.js中，这些方法只能抛出断言错误(assertion errors)。
2. 在先前版本的Node.js中，如果定时器的延时溢出，不会提供任何溢出发生的指示，而在Node.js 9中，定时器会发出警告。
3. `NODE_OPTIONS` 新增了 `stack-trace-limit` 属性，用于开发环境下设置堆栈上限，使用方式:`NODE_OPTIONS=--stack-trace-limit=100`
4. 支持 `console.debug` 方法，此方法与`console.log`表现一致
5. `cluster.settings`中允许通过`cwd`属性配置目标目录
6. 在使用Electron等程序时，我们需要控制什么时候使用v8 platform，为了使Node正常工作，我们有时需要手动创建NodePlatform。Node.js 9 新增了用于创建/销毁NodePlatform的公共API
7. stream模块中，新增了 `state.ending` 属性判断stream是否已调用 `end()` 方法
8. `async_wrap`新增了两个属性值：`TCPSERVERWRAP` 和 `PIPESERVERWRAP`，从而允许我们通过连接种类区分服务器
9. 添加了新的异步钩子，提供一个API来注册回调从而跟踪应用程序中的所有异步资源，记录其所观察到的异步操作的时间信息
10. 在[N-API（用于构建本地插件的API）](https://nodejs.org/api/n-api.html#n_api_n_api)中，Node.js 9中为需要引用当前事件循环的插件添加了一个函数。从而使插件得以访问当前的事件循环，要注意的是这个特性目前仍是实验性的



