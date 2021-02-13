---
title: PWA+ 老板再也不用担心我的性能优化KPI  (报告篇)
date: 2019-10-22 15:14:44
tags: 
  - PWA
  - 性能优化
---

> 结论先行：在目前的默认构建下，PWA+SSR的最佳实践是 PWA (html+静态资源) + 离线包 ，即同时启用 PWA 和离线包。利用离线包的能力覆盖首次用户，利用 PWA 的能力覆盖二次用户。 在本项目中，采用上述方案，页面首屏时间提升了20% (440ms)，可交互时间提升了24% (710ms)

## 背景

之前团队内已经由两位输出了 PWA+非直出 方案的接入报告，而 PWA+直出 的方案一直缺乏落地数据沉淀。近期我们在Now APP 附近页面尝试接入了 PWA+直出 方案，并利用控制变量法和离线包等方案做了对比，这里我整理了接入效果对比和数据分析，并总结接入心得和问题，供大家参考。

接入项目：NOW APP内附近页
应用场景：NOW独立版一级页面
接入方案：PWA+SSR（PWA+直出渲染）

![img](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/1571727027_25_w480_h1040.gif)

<!-- more -->

## 接入方案

需要指出的是，本次实验接入的是 PWA+SSR 方案，即 PWA+服务端渲染方案。主要是利用线上直出 HTML 中的 state 来更新旧的缓存 HTML 页面。

为了获取更可靠的数据，同时也为了对比 PWA+ 的缓存策略和离线包的方案，我们的接入方案分为四个阶段：

1. 第一阶段(27/9 - 8/10)：PWA（html+js）离线包- ：这一阶段我们只开启了 PWA，用于缓存静态资源和html主文档
2. 第二阶段(8/10- 14/10)：PWA（html+js） 离线包+：这一阶段我们同时启用了PWA和离线包
3. 第三阶段(14/10- 17/10)：PWA（html）离线包+：这一阶段我们依然启用了PWA和离线包，但只利用了PWA缓存html主文档
4. 第四阶段(17/10-21/10)：PWA- 离线包+：这一阶段我们关闭了PWA，只启用了离线包

利用 PWA 配置平台，我们总共只进行了一次发布，此后的 PWA 开关和缓存配置修改都在平台上进行。

通常我们的项目采用的是第四阶段的方案，也就是只接入了离线包，我们用第四阶段的方案作为基准对比。

## 接入前后效果对比

*由于 now app内, ios不支持 PWA ，以下所有数据都来源于安卓端*

这里挑选几个重点关注的测速进行分析，更详细的数据可以查看我们的 issue(数据存放在内网，此处就不展示了)

### 首屏耗时

|            | 第一阶段 | 第二阶段 | 第三阶段 | 第四阶段 |
| ---------- | -------- | -------- | -------- | -------- |
| 首屏耗时   | 1750ms   | 1660ms   | 1660ms   | 2080ms   |
| 提升百分比 | 15.9%    | 20.2%    | 20.2%    | -        |
| 可交互时间 | 2380ms   | 2230ms   | 2420ms   | 2940ms   |
| 提升百分比 | 19.0%    | 24.1%    | 17.7%    | -        |

![img](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/image0.png)

![img](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/image1.png)

从测速数据来看，接入效果最好的是 第二阶段 ，即 PWA+(缓存HTML主文档和静态资源)+离线包方案，与只接入离线包方案相比，首屏时间提升了20% (440ms)，可交互时间提升了24% (710ms)

下面是我们针对这组数据的一些分析：

1. 首屏耗时和可交互时间的区别是什么？
   - 首屏耗时是用户可见页面数据的时间（直出的情况下是HTML加载的时间，非直出的情况下是首屏 cgi 请求返回的时间），可交互时间是指页面可响应交互，通常是 js 加载执行完毕的时间。
2. 为什么第二阶段（PWA+离线包）比单纯PWA的速度更快？
   - 离线包可以覆盖首次用户，而 PWA 只能覆盖二次用户。
3. 如何解释第三阶段和第二阶段的数据对比？
   - 这两个阶段中，唯一不同是 PWA 是否缓存了静态资源。二者首屏耗时是一样的，这很容易理解。而可交互时间上，阶段三比阶段二慢了约100ms，这是否说明从离线包中读取静态资源比从从 sw 中读取静态资源要慢呢？答案并非如此，事实上，我们在现网 HTML 上通过 script 标签插入的 js，src 都带上了 _bid=152, 这是一个不存在的 bid，无法命中离线包。所以在第三阶段，命中 sw 缓存 html 的静态资源只能走线上和 browser cache，自然比第二阶段要慢。
4. 是否能利用 PWA 缓存 HTML 并利用离线包缓存静态资源？
   - 同上，需要修改构建，带上正确的 bid。
5. PWA 缓存静态资源 , 离线包缓存 和 browser cache 哪个快？
   - 目前数据还不清楚，后续我们会继续研究

### 页面启动

|            | 第一阶段 | 第二阶段 | 第三阶段 | 第四阶段 |
| ---------- | -------- | -------- | -------- | -------- |
| page_start | 1280ms   | 990ms    | 990ms    | 750ms    |

![img](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/image2.png)

`page_start` 这个标识位打在 html 的 `<head>` 中，很显然，随着离线包的比例增加，page_start 的速度也越来越快，从中我们得出结论：

1. 从离线包里获取 html 的速度要比 从 service worker 中 获取缓存的 html 更快，我们认为这是由于离线包借助了客户端能力，在打开 webview 的同时就能去获取离线包，而 service worker 需要等 webview 打开后才能注册/激活执行。

### 二次用户率

我们通过 monitor 上报了安卓用的 pv 以及安卓的二次用户 pv。通过四次抽样调查，我们发现安卓的二次用户率 在 **80%** 左右。

### Service Worker上报

|                           | 第一阶段 | 第二阶段 | 第三阶段 | 第四阶段 |
| ------------------------- | -------- | -------- | -------- | -------- |
| sw HTML命中率（二次用户） | 72.1%    | 65.4%    | 65.0%    | 0%       |
| sw HTML命中率（全部用户） | 57.2%    | 51.4%    | 51.9%    | 0%       |
| 离线包命中率（全部用户）  | 0%       | 45.1%    | 45.3%    | 99.4%    |

在第一阶段时，sw HTML命中率（二次用户）为 72.1%，这与之前 v 同学邮件中 sw 支持率 (75%) 相当。

在第二三阶段时，sw HTML命中率（二次用户）为 65%，对比前一阶段，有 10% 左右的用户二次进入时，未命中sw，推测这部分用户可能被离线包分流了，而后续sw的命中率一直很稳定（65%以上），我们推测离线包和 sw 的优先级和内核版本有关，后续也会持续跟进研究。

## 接入心得

PWA-PLUS的接入不是很复杂，具体接入方式可以参考[PWA-PLUS: 老板再也不用担心我的性能优化KPI （1.0 方案篇）](https://sangle7.com/2019/09/18/PWA+:%20%E8%80%81%E6%9D%BF%E5%86%8D%E4%B9%9F%E4%B8%8D%E7%94%A8%E6%8B%85%E5%BF%83%E6%88%91%E7%9A%84%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96KPI%20%20(%E6%96%B9%E6%A1%88%E7%AF%87)/)。这里不再赘述。

这个项目本来就是 SSR 的，这次接入主要要做几件事：

1. 项目中添加`pwa_config.json`, 此文件用户用于灰度 PWA 配置

   ```javascript
   {
       "id": 86168,
       "is_gray": true,
       "gray_uins": "523374254",
       "gray_regex": ""
   }
   ```

2. 构建引入 @tencent/pwa-plus-plugin

```javascript
// feflow.js
const PWAPlusPlugin = require('@tencent/pwa-plus-plugin');
// ...
new PWAPlusPlugin({
    swDest: 'webserver/nearbyjiaoyou/sw.js',
    dev: isFeflowDev, 
    pid: 86168, 
    cacheName: 'now-app-nearbyjiaoyou', 
    web: '//now.qq.com/', 
    cdn: isFeflowDev ? '//now.qq.com' : '//11.url.cn/', 
    product: isFeflowDev ? '' : 'now',
    moduleName: isFeflowDev ? '' : 'app',
    bizName: isFeflowDev ? '' : 'nearbyjiaoyou',
    cacheFirst: [/\.(js|css)$/],
    staleWhileRevalidate: [
        {
            test: /nearbyjiaoyou\/.*\.html/,
            ignoreSearch: true
        }
    ],
    ssrReg: [
        {
            test: /nearbyjiaoyou\/.*\.html/,
            ignoreSearch: true
        }
    ],
    include: [/\.(js|css)/],
    exclude: []
})
```

*HTML 资源的缓存写入和匹配，通过设置 ignore: true 忽略参数 (NOW独立版场景，客户端打开页面每次拼接的时间戳参数_t均不同，不忽略参数会导致每次都没法命中缓存）*

1. RootComponent 引入 `@tencent/pwa-plus-helper`，并通过 `wb.queryRequest` 方式上报 js 命中。

```javascript
 const { Workbox } = require('@tencent/pwa-plus-helper');

wb = new Workbox('./sw.js', { scope: './', id: 86168 });


const scriptUrl = document.currentScript && document.currentScript.src;

const registerRes = wb.register();

if (registerRes && registerRes.then) {
        registerRes
            .then(() => {
                if (scriptUrl) {
                    // sw 命中 js 上报
                    wb.queryRequest(scriptUrl)
                        .then(({ payload }) => {
                            if (payload) {
                            	monitor.report(34713150);
                            	}
                            })
                        .catch(() => {});
                    }
                })
            .catch(err => {
        console.error(err);
    });
}
```

1. 在 RootComponent 中，判断一下 window.__initialstate 是否存在，如果存在则需要和 sw 通信，获取新的页面 state，并调用更新方法：

```javascript
componentDidMount() {
    if (!shoudlLoadData) { 
        this.fetchFromSW();
    }
}

fetchFromSW = () => {
    const { dispatch } = this.props;

    // 从 service worker 获取最新数据
    wb.queryState()
        .then(({ payload = {} }) => {
        // payload 为获取到的最新数据
        // 然后调用更新方法来更新页面数据
        console.log('payload', payload);

        const { data = '' } = payload;

        const state = JSON.parse(data);

        dispatch({
            type: 'MODIFY_STATE',
            data: state.jiaoyouList
        });
    })
        .catch(err => {
        console.error(err);
    });
};
```

1. 上报安卓PV和安卓二次用户
2. 由于这个页面和地理位置有关，直出的时候使用的是客户端种在url上的location，可能会有获取不到的情况，直出后需要调用 jsbridge 更新一下位置信息

## 总结和规划

### 总结

1. 在目前的默认构建下，PWA+SSR的最佳实践是 PWA(html+静态资源) + 离线包 ，即同时启用 PWA 和离线包。利用离线包的能力覆盖首次用户，利用 PWA 的能力覆盖二次用户。
2. 在同时开启 PWA 和离线包的方案下，安卓二次用户的 sw 命中率在 65% 左右，这是一个相当高的命中率。
3. 页面同时也接入了 aegis 测速 ，但由于 aegis 还未支持自定义打点上报，同时也没有区分操作系统，本次测速数据还是采用了 wang 的数据。
4. 接入时开发阶段的调试成本比较高，后续我们会想办法降低调试成本。

### 规划

1. 该页面的二级嗨玩页，可接入二级页面缓存方案，提前注册service worker 并写入缓存。
2. 沉淀 SSR 项目接入 PWA 方案 SOP，快速推进 NOW APP 内的旧页面接入。
3. 继续研究 PWA 缓存静态资源 , 离线包缓存 和 browser cache的测速数据
4. 继续研究 sw 和离线包优先级与内核版本的关系