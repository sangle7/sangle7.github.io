---
title: 了解 JavaScript 中的 Memoization 以提高性能
date: 2019-02-05 19:50:44
tags: 
  - 翻译
---


如何通过更好的 Javascript memoization 提高您的应用程序性能



我们渴望提高应用程序的性能。 Memoization 是 JavaScript 中的一种技术，通过缓存结果并在下一个操作中使用缓存来提高速度。

在本文中，我们将看到 memoization 的用法以及它如何优化应用的性能。 



## Memoization: 基本理念

如果我们有 CPU 密集型操作，我们可以通过将初始操作的结果存储在缓存中来优化。 如果操作必须再次执行，我们将不必再次使用我们的 CPU，而只用简单地返回结果。

它可以表示如下：

```javascript
function longOp(arg) {
    if( cache has operation result for arg) {
        return the cache
    }
    else {
        perform the long operation, takes 30 minutes
        store the result in cache
    }
    return the result
}
longOp('lp') // performs the operation because it is the first time, it takes 30 minutes
// Next, it stores the result in cache
longOp('bp') // performs the operation because it is the first time for this set of inputs `bp`, it takes 30 minutes
// Next, it stores the result in cache
longOp('bp') // same operation
// It just returns the cache without perfomrming the long operation, takes 1 second
longOp('lp') // same operation
// It just returns the cache without perfomrming the long operation, takes 1 second
```

<!-- more -->

就 CPU 使用而言，上述伪函数 longOp 是一种昂贵的功能。 首先，它允许操作针对特定的一组输入运行并缓存操作的结果。 在具有相同输入的后续调用中，它仅返回先前存储在缓存中的结果，绕过时间和资源消耗操作。

使用args lp的第一次调用是耗时的操作，需要 30 分钟才能完成。 当使用相同的输入 lp 调用函数时，它会检索存储在缓存中的先前结果，这大约需要。 1秒。 输入 bp 的情况也是一致的。

使用真实示例，假设我们有一个函数可以找到数字的平方根：

```javascript
function sqrt(arg) {
    return Math.sqrt(arg);
}
log(sqrt(4)) // 2
log(sqrt(9)) // 3
```

我们可以记住这个 sqrt 函数：

```javascript
function sqrt(arg) {
    if (!sqrt.cache) {
        sqrt.cache = {}
    }
    if (!sqrt.cache[arg]) {
        return sqrt.cache[arg] = Math.sqrt(arg)
    }
    return sqrt.cache[arg]
}
```

我们现在看到 sqrt 函数将每个结果保存在作为函数属性的缓存对象中。

请注意，在记忆功能时，首先创建缓存并检查输入是否已在缓存中，如果不在则继续执行操作。

我们可以多次调用该函数：

```javascript
sqrt(9)
sqrt(9)
sqrt(4)
```

使用9作为参数的第一次调用不会被缓存，进行计算并缓存结果。

第二次调用返回缓存的结果。

第三次调用将执行计算，因为输入不存在于缓存中。行计算并缓存结果，以便使用相同的输入下一次调用函数。

我们可以添加日志来查看存储在缓存中的对象：

```javascript
//...
log(sqrt.cache) // { "9": 3, "4": 2 }
```

另一个例子，假设我们有一个平方函数来计算传递给它的数字的平方。

```javascript
function square(num) {
    return num * num;
}
log(square(2)) // 4
log(square(4)) // 16
```

我们可以这样记住这个函数：

```javascript
function square(num) {
    if(!square.cache) {
        square.cache = {}
    }
    if(!square.cache[num]){
        return square.cache[num] = (num * num)
    }
    return square.cache[num]
}
```

与sqrt函数相同，它检查输入是否已经有结果。 如果是，则返回缓存中的结果。 如果不是，则执行计算，返回结果并将结果存储在缓存中。

```javascript
log(square.cache) // undefined, no cache initially
log(square(2)) // 4
log(square.cache) // { "2": 4 }, the result of input 2 now in cache
log(square(2)) // 4
// Simply, returns the result of the input 2 stored in the cache from the above operation.
log(square(4)) // 16, does the operation becuase no operation with input 4 has been carried out before
log(square.cache) // { "2":4, "4": 16}, now input 4 result is now in cache.
log(square(4)) // 16, you can guess it retrievs theresult from the cache
```



## Memoization: 执行

在上一节中，我们更改了函数，以便为它们添加 memoization。

现在，我们可以创建一个独立的函数来记忆任何函数。 我们将此函数称为 memoize。

```javascript
function memoize(fn) {
    return function () {
        var args =
Array.prototype.slice.call(arguments)
        fn.cache = fn.cache || {};
        return fn.cache[args] ? fn.cache[args] : (fn.cache[args] = fn.apply(this,args))
    }
}
```

我们看到这个函数接受另一个函数作为参数并返回一个函数。

要使用此函数，我们调用memoize将要记忆的函数作为参数传递。

```javascript
memoizedFunction = memoize(funtionToMemoize)
memoizedFunction(args)
```

例如，让我们执行我们的第一个例子

```javascript
function sqrt(arg) {
    return Math.sqrt(arg);
}
const memoizedSqrt = memoize(sqrt)
```

返回的函数 memoizedSqrt 现在是 sqrt 的 memoized 版本。

Let’s try it:

```javascript
//...
memoizedSqrt(4) // 2 calculated
memoizedSqrt(4) // 2 cached
memoizedSqrt(9) // 3 calculated
memoizedSqrt(9) // 3 cached
memoizedSqrt(25) // 5 calculated
memoizedSqrt(25) // 5 cached
```

我们可以将 memoize 函数添加到 Function 原型中，以便我们的应用程序中定义的每个函数都继承 memoize 函数并可以调用它。

```javascript
Function.prototype.memoize = function() {
    var self = this
    return function () {
        var args =
Array.prototype.slice.call(arguments)
        self.cache = self.cache || {};
        return self.cache[args] ? self.cache[args] : (self.cache[args] = self(args))
    }
}
```

我们知道 JS 中定义的所有函数都是从 Function.prototype 继承的。 因此，添加到 Function.prototype 的任何内容都可用于我们定义的所有函数。

让我们看看它的实际效果：

```javascript
function sqrt(arg) {
    return Math.sqrt(arg);
}
// ...
const memoizedSqrt = sqrt.memoize()
log(memoizedSqrt(4)) // 2, calculated
log(memoizedSqrt(4)) // 2, returns result from cache
log(memoizedSqrt(9)) // 3, calculated
log(memoizedSqrt(9)) // 3, returns result from cache
log(memoizedSqrt(25)) // 5, calculated
log(memoizedSqrt(25)) // 5, returns result from cache
```

## Memoization: 速度和测试

Memoization 的目标是提高速度。 Memoization 消耗内存空间以提高速度。 在爱立信消费者实验室进行的分析中，发现等待加载慢速网页与观看恐怖电影相当。（？？？）

通过 memoization，我们可以加快应用程序的性能。

显示执行非记忆功能和记忆的时间差异。 我们将使用Node提供给我们的tie阅读方法来测试我们的功能。

为了展示未添加 memoization 功能的函数和添加 memoization 的函数执行时间差异，我们使用 Node 提供的tie阅读方法来测试。

我们对 sqrt 函数进行了测试：

```javascript
function sqrt(arg) {
    return Math.sqrt(arg);
}
const memoizedSqrt = memoize(sqrt)
console.time("non-memoized call")
console.log(memoizedSqrt(4))
console.timeEnd("non-memoized call")
console.time("memoized call")
console.log(memoizedSqrt(4))
console.timeEnd("memoized call")
$ node memoize
2
non-memoized call: 8.958ms
2
memoized call: 1.298ms
```

我们看到非 memoized 函数 sqrt 比 memoized 函数运行得慢。 当然这是因为 memoized 函数不计算已经缓存的相同输入，而是返回先前存储的结果。

在我的机器上运行了很多次测试，memoized 函数总是优于非 memoized 函数：

```shell
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
2
non-memoized call: 8.215ms
2
memoized call: 0.566ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
2
non-memoized call: 7.062ms
2
memoized call: 0.354ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
2
non-memoized call: 7.300ms
2
memoized call: 0.303ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
2
non-memoized call: 7.681ms
2
memoized call: 0.301ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
2
non-memoized call: 7.170ms
2
memoized call: 1.225ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$
```

接下来，我对 `square` 方法进行了测试

```shell
function square(num) {
    return num * num;
}
const memoizedSqr = memoize(square)
console.time("non-memoized call")
console.log(memoizedSqr(2))
console.timeEnd("non-memoized call")
console.time("memoized call")
console.log(memoizedSqr(2))
console.timeEnd("memoized call")
```

获得如下的结果：

```
$ node memoize
4
non-memoized call: 9.455ms
4
memoized call: 4.567ms
```

和上面的例子一致，memozied 的函数更快。

更多的测试：

```shell
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
4
non-memoized call: 7.167ms
4
memoized call: 0.294ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
4
non-memoized call: 7.444ms
4
memoized call: 0.562ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
4
non-memoized call: 7.472ms
4
memoized call: 1.487ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
4
non-memoized call: 7.002ms
4
memoized call: 0.790ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
4
non-memoized call: 7.841ms
4
memoized call: 1.788ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$
```

## Memoization: 什么时候应该用？

在这里，memoization 通常会缩短执行时间并影响我们应用程序的性能。 当我们知道一组输入将产生某个输出时，memoization 最有效。

遵循最佳实践，应该在纯函数上实现 memoization。 纯函数是指输出仅取决于输入而不产生负作用的函数。

由于 memoization 以空间换速度，因此应在具有有限输入范围的函数中使用 memoization，以便更快地进行检查。

在处理递归函数时，Memoization 最有效，递归函数用于执行诸如 GUI 渲染，Sprite 和动画物理等繁重操作。

## Memoization: 什么时候不该用？

当函数的输出不依赖于输入以及输出随时间变化时，不应使用 memoization 。

我们来看看这个例子：

```javascript
function unpredicatble(input) {
    return input * Date.now()
}
log(unpredicatble(4))
log(unpredicatble(4))
$ node memoize
6169843078712
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
6169843091424
```

你看，函数是用相同的参数调用的，但却产生了不同的输出。 输出会随着时间的推移而变化，并且它不适合进行记忆，因为记忆不会捕获当前输出。

## 使用案例：斐波那契数列

斐波那契是许多复杂算法中的一种，可以使用 memoization 进行优化。

```javascript
1,1,2,3,5,8,13,21,34,55,89
```

斐波那契系列的特点是前两个之后的每个数字都是前两个数字的总和。

 ```javascript
F2 = F1 + F1 => (2 = 1+1)
F5 = F3 + F2 => (5 = 3+2)
Therefore,
Fn = Fn-1 + Fn-2
 ```

在 JS 中，我们可以这样表示：

```javascript
function fibonacci(num) {
    if (num == 1 || num == 2) {
        return 1
    }
    return fibonacci(num-1) + fibonacci(num-2)
}
```

如果 num 超过2，则此函数是递归的。

```javascript
log(fibonacci(4)) // 3
```

让我们对 memoized 后的斐波那契算法进行测试。

```javascript
const memFib = memoize(fibonacci)
```

```javascript
log('profiling tests for fibonacci')
console.time("non-memoized call")
log(memFib(6))
console.timeEnd("non-memoized call")
```

```javascript
console.time("memoized call")
log(memFib(6))
console.timeEnd("memoized call")
```

```bash
$ node memoize
profiling tests for fibonacci
8
non-memoized call: 2.152ms
8
memoized call: 1.270ms
```

第一次运行用了 2.152ms，第二次命中了缓存，只用了1.270ms。节省了 0.882ms ，很大一笔时间。

更多的测试如下：

```bash
$ node memoize
profiling tests for fibonacci
8
non-memoized call: 3.031ms
8
memoized call: 1.136ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
profiling tests for fibonacci
8
non-memoized call: 1.444ms
8
memoized call: 0.249ms
Admin@PHILIPSZDAVIDO MINGW64 /c/wamp/www/developerse/projects/trash
$ node memoize
profiling tests for fibonacci
8
non-memoized call: 1.200ms
8
memoized call: 0.242ms

```

你看到在 memoized 的函数中，有些时候的执行几乎是 0ms。

## Conclusion

在这篇文章中，我们了解了 memoization 的作用以及它如何影响我们的 Web 应用程序的性能。

接下来，我们继续创建了几个函数和它们的 memoized 版本。 然后，我们对它们进行了基准测试，发现 memoized 后的版本的性能要优于 未memoized 的版本。 最后，我们探讨了几个用例，以说明 memoization 如何对它们产生巨大影响。

尽管 memoization 有很大的好处，但它带来了成本。 它以大量内存使用来换取速度。 这在在低内存函数中不会引起问题，但在处理 RAM 密集型函数时会产生影响。
