---
title: PWA-PLUS 3.0 - 提高缓存命中率的秘诀！
date: 2020-10-22 15:14:44
tags: 
  - PWA
  - 性能优化
---

> 接入中心化 service worker 的项目，用户每次访问一个页面都会缓存关联的其他页面资源，为其他页面的载入加速，我愿称之为 PWA 届的 P2P （手动推眼镜.jpg


## 写在最前

不知不觉距离上一篇PWA相关文章已经过了半年了，这段时间主要在~~摸鱼~~研究如何提升缓存命中率的问题。

在上一篇报告中我们提到两点：

1. 旧方案的二次用户 service worker 命中率有限
2. 尝试在一级页面去加载二级页面的缓存资源

针对这两个问题/设想，我们做了一些尝试，主要目的在于提升 service worker 的缓存命中率，从而提升首屏速度：

<!-- more -->


## 预缓存？

最初的设想是在 A 页面的 sw 中提前缓存 b 页面的缓存资源，不过这个方案很快就被否定了，原因是在我们已有的pwa-plus方案中，每个页面都是单独注册各自的 service worker，举个栗子：

A页面：

<img src="https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587887992_18_w756_h1336.png" width="200" />

点击红框部分即可打开B页面：

<img src="https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587888004_60_w752_h1340.png" width="200" />

A页面注册的sw路径 -> now.qq.com/a/sw.js （假的，不要点）

B页面注册的sw路径 -> now.qq.com/b/sw.js

好的，很显然此时我们在A页面上缓存的资源都进了 a/sw.js，并且该 sw 只能控制 now.qq.com/a/ 下的内容，自然也就无法控制B页面的响应。

基于这个失败的设想，我们很快就想到了，既然预缓存的方案行不通，那是否可以在A页面提前注册B页面的sw呢？

## 预注册？

我们的设想是这样的：在A页面去预注册B页面的 service worker 并拉取缓存，这样用户在首次进入B页面的时候，就能命中sw缓存。

还是以上面的AB页面路径为例，我们在A页面项目的代码中加了这么一行

```js
const wb = new Workbox('/sw.js', { scope: '/', id: 86168 }); //A页面原有的sw
const wbB = new Workbox('//now.qq.com/b/sw.js', { scope: '/', id: 86173 }); //注册B页面的sw
```

（当然前提是AB两个页面都已经接入了pwa-plus方案）

这样在用户打开A页面时，会同时注册a/sw.js 和 b/sw.js 两个service worker。同时 b/sw.js 会去加载B页面的缓存列表并进行缓存，看起来没有大的问题，线上数据也能看出来是生效的。

但如果A页面有很多个二级页呢？按照这个方案，我们需要写成这样：

```js
const wb = new Workbox('/sw.js', { scope: '/', id: 86168 }); //A页面原有的sw
const wb1 = new Workbox('../b/sw.js', { scope: '/', id: xxx }); //注册B页面的sw
const wb2 = new Workbox('../c/sw.js', { scope: '/', id: xxx }); //注册C页面的sw
const wb3 = new Workbox('../d/sw.js', { scope: '/', id: xxx }); //注册D页面的sw
const wb4 = new Workbox('../e/sw.js', { scope: '/', id: xxx }); //注册E页面的sw
const wb5 = new Workbox('../f/sw.js', { scope: '/', id: xxx }); //注册F页面的sw
...
```

看起来是不是很不优雅，并且每一个二级页面的增加都需要修改A这个入口项目的代码。

有没有一种方法，在增加子页面的时候，只需要在平台上修改一些配置，而不用修改项目代码呢？

于是我们又提出了中心化sw的想法：

## 中心化？

中心化sw，顾名思义就是让 sw.js 文件位于 AB 页面的公共路径上，以上面的 AB 页面为例，我们可以在 now.qq.com/sw.js 部署 sw，从而可以同时控制 AB 路径。

### 方案设计

[PWA-PLUS: 老板再也不用担心我的性能优化KPI （1.0 方案篇）](https://sangle.info/2019/09/18/PWA+:%20%E8%80%81%E6%9D%BF%E5%86%8D%E4%B9%9F%E4%B8%8D%E7%94%A8%E6%8B%85%E5%BF%83%E6%88%91%E7%9A%84%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96KPI%20%20(%E6%96%B9%E6%A1%88%E7%AF%87)/) **如果你不了解原方案，强烈建议先看下原方案**

旧方案的架构图：

![img](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1568799755_86_w1271_h749.png)

新方案的架构图长这样：

![enter image description here](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587888342_5_w1211_h551.png)

主要区别在于两点：

1. 将每个项目单独注册的sw换成了中心化的通用sw
2. 返回资源列表时，同时返回相关项目的缓存列表

### 方案实现

这部分是具体的方案实现过程，不关心实现逻辑的同学可以直接跳到项目迁移部分

可以在这个issue跟踪 https://git.code.oa.com/pwa-plus/pwa-plus-plugin/issues/25

实现这个方案也经历了一些波折，方案主要的难点是，在接入pwa-plus方案时，项目会绑定一个唯一的id，pwa-plus平台也会根据这个项目id来下发配置列表。

此前，在注册sw的时候，我们会传入项目id：

```js
const wb = new Workbox('/sw.js', { scope: '/', id: 86168 }); //A页面原有的sw
```

对应的sw代码是这样的

```js
export default class Precache {
  constructor (id, options) {
    this._id = id
    this._cacheName = options.cacheName || `pwa-plus-${id}`
    this._options = options
    this._cache = new PrecacheCache(this._cacheName)

    this._active().catch(console.error)
  }
  //....
}
```

最初我们的设想是，可以让sw从query中获取 id，注册时用这样的写法：

```js
const wb = new Workbox('/sw.js?id=86168', { scope: '/' }); //A页面代码
const wb = new Workbox('/sw.js?id=86173', { scope: '/' }); //B页面代码
```

设想很棒！然而…

尝试后发现，不同 query 的 sw.js 会注册不同的 service worker，无法共享同一个实例。

<img src="https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/lg_1301377_1545139756_5c18f62cb4aa3.jpeg" width="200" />

query 和 hash 都不行，似乎没有方法能够在注册时同步传数据给sw。

---

问：为什么要同步传数据给 sw？异步传不行吗？

答：这个问题要分为两部分考虑，在 pwa-plus 方案里，我们将 cache 分为两部分：precache 和runtime，其中 precache 是以 pwa-plus id为key，缓存此id对应的文件列表

![enter image description here](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587888054_74_w2142_h634.png)

而 runtime 是针对 HTML 缓存的，里面是通过 SSR/CSR 方案缓存的 HTML 资源

![enter image description here](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587888062_99_w2524_h696.png)

其中，precache 只和 id 有关，我们改造了一下，添加了一个 setId 的方法，允许异步传入 id，同时，由于 id 是异步传入的，为了在 id 还没传入时能够匹配上资源，修改了匹配 cache 的方法：

 ```js
 // 新：从所有cache中遍历
 const responseHandle = caches.match(request, { ignoreSearch: true })
 
 // 旧：从指定id的cache中寻找
 const responseHandle = this._cache.matchOrFetch(request, { ignoreSearch})
 ```

然而，runtime 中的缓存是用于 HTML 的，为了能命中 HTML，无法使用异步的方式传入（异步传入要在js中执行，必然晚于html的加载）

最后灵机一动(?)，采取了类似织云配置下发的方式，每次有新配置时都重新下发根路径下的 sw 文件


### 项目迁移

迁移到新方案需要做四件事

1. 添加项目关联关系

   访问 platform.pwa.oa.com 管理平台，添加关联项目，当用户访问当前项目的页面时，sw会拉取所有关联项目的缓存列表

![enter image description here](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1587888092_53_w2064_h882.png)

2. 升级sdk, 修改代码中注册sw的路径，并手动调用setid

升级 @tencent/pwa-plus-helper 和 @tencent/pwa-plus-plugin 到 0.8.1-alpha.0 版本

已接入pwa-plus方案的项目，只需要在sw注册成功后手动 setid 即可：

```js
// A页面
const wb = new Workbox('https://now.qq.com/pwaplus_sw.js', { scope: '/' });
const registerRes = wb.register();
registerRes.then(() => {
  wb.setId(86168);
})
```

3. 如果页面需要接入HTML缓存，还需要修改 https://now.qq.com/pwaplus_sw.js 文件的配置

   此处会在未来提供一个修改的接口，项目发布后调用接口即可，目前如果需要修改此文件，可以先联系 @sanglewang 协助



### 接入数据

接入中心化方案后：

|                  | 旧方案（独立sw）    | 接入后                            |
| ---------------- | ------------------- | --------------------------------- |
| 一级页面sw命中   | 55%（二次用户）     | 55%（二次用户）                   |
| 一级页面首屏时间 | 1660ms              | 1550ms                            |
| 二级页面sw命中   | 13%（二次用户 40%） | 48%（首次用户 20%，二次用户 90%） |

*这个二级页面的二次用户占比很低，只有20%左右

可以看到，二级页面的二次用户命中率达到了90%

## 总结和规划

### 总结

1. 在预加载，预注册和中心化sw的方案中，最有效且通用的是中心化sw方案。
2. 中心化sw方案的核心是利用一个公共路径下的sw.js文件，控制所有子路径的项目缓存。
3. 中心化sw方案通过预加载其他页面的资源，提升缓存命中率，同时也支持缓存 html 文档，和已有的 html 缓存方案不冲突，可同时使用。
4. 在接入中心化sw后，二级页面的二次用户命中率达到了90%以上。
5. 已接入PWA-PLUS的项目，迁移成本很低。我们也会尽快整理出一份迁移指南。
6. ~~接入中心化sw的项目，用户每次访问一个页面都会缓存关联的其他页面资源，为其他页面的载入速度加速，我愿称之为PWA的P2P~~

### 规划

1. 快速推进 NOW APP 内的旧页面接入中心化方案
2. 提供一个修改的接口，项目发布后调用接口即可修改主sw.js文件配置，无需手动修改。
3. 持续跟进二级页面首次命中不高的原因
