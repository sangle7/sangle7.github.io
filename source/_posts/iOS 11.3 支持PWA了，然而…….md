---
title: iOS 11.3 支持PWA了，然而……
date: 2018-02-08 18:10:44
tags: 
    - PWA
    - iOS
    - 翻译
---

原文：[https://medium.com/@firt/pwas-are-coming-to-ios-11-3-cupertino-we-have-a-problem-2ff49fd7d6ea](https://medium.com/@firt/pwas-are-coming-to-ios-11-3-cupertino-we-have-a-problem-2ff49fd7d6ea)

![](http://onvaoy58z.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-02-09%20%E4%B8%8B%E5%8D%885.44.15.png)

1月25日，Ricky Mondello 发表了一篇推特，随后的 [safari 11.1 beta 更新说明](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_11_1.html)中也提到了Web App Manifest  和 Service Workers ，这意味着跨平台的PWA 成为了现实，让我们来看看究竟有什么变化

<!-- more -->

**2月8日更新**：beta2已经发布了，webkit博客也发布了[一篇关于Service Workers这个平台上的运行机制的博客](https://webkit.org/blog/8090/workers-at-your-service/)，基于这篇博客，本文章更新了一些内容。



## 测试并不容易

在 iOS 上进行这些新功能的测试并不容易，因为 Safari 上的开发者工具目前还不能让我们看到iOS 上的 Service Workers， 除此之外，允许我们与 Service Worker 通信的 MessageChannel在 iOS 上也还不存在。尽管如此，我还是研究了好几个小时，除了WebKit 团队肯定会在最终版本中修复的一些未通过单元测试的 bug，我将重点放在 iOS PWA 和 Android 上的 PWA 的对比上。

> 如果你已经发布了 PWA 或是即将发布，你应该关注它们在 iOS 上可能会出现的一些问题

18个月之前（居然已经一年半了？），我发布了[**Don’t use iOS meta tags irresponsibly in your Progressive Web Apps**](https://medium.com/@firt/dont-use-ios-web-app-meta-tag-irresponsibly-in-your-progressive-web-apps-85d70f4438cb)**.** 一些公司（例如 Twitter 和Flipkart ）在那时候关注到这些问题并移除了iOS上的meta标签或是解决了这些问题。

那时的问题是一些公司选择通过苹果 meta 标签来支持 iOS 的主屏幕，它们甚至没有进行测试，也没有意识到 Android 的 PWA 和 iOS 之间的差异。

很遗憾的是，大部分问题与我 18 个月前所说的相同，只是有一个大的不同：现在你不需要通过标签选择加入 iOS;  iOS 将添加对 Web App Manifest 的支持，所以你的 PWA 将自动成为 iOS PWA。 但是苹果在 PWA 的行为上和 Android 并不一致，这意味着：Cupertino，我们有麻烦了。

## 只能在线使用的已安装程序

如果你将 PWA 安装到了主屏幕，那么Service Workers 将被注册，但它并不处于工作状态，这是因为 Service Workers 并不会将 web app 作为客户端，所以不要期望在第一个 beta 测试版中能获得离线的体验，但我认为这应该是一个会在未来的 beta 版本中的得到修复的 bug

![](http://onvaoy58z.bkt.clouddn.com/1_x5NLsHEsNaeqfu6yW3upFA.png)

## 一个完全不同的 Service Workers

*这一段添加于2月8号*

苹果添加了 Service Workers APi，但苹果对于它的运行机制，以及我们可以用它做什么的想法都和以前不同，最大的不同在于：

> “为了仅保留对用户有用的存储信息，WebKit 将在几周后删除未使用的 Service Worker 注册。 几周后不会打开的缓存也将被删除。 Web 应用程序必须能够正确应对单独缓存，缓存条目以及 Service Worker 被移除的情况。“
>
> https://webkit.org/blog/8090/workers-at-your-service/

**这是一个巨大的改变**！在 Chrome, Firefox, Samsung Internet 以及其他浏览器中，Service Worker 的注册将永远不会被自动移除，我们可以在将来用到他们，这正式为什么安装PWA能允许我们在将来离线运行。但根据苹果的说法，Service Worker和缓存在将来是否可用是没有保证的。用户有可能在“几个星期后“回到 web 应用程序。我知道，web 应用程序在在线时仍然可以工作，但这样我们无法保证PWA的一个关键概念：离线访问。

虽然从浏览器的角度来看似乎是公平的，但我并不认为对于那些用户放在主屏幕中的 PWA 是公平的。

在“几个星期后”后的苹果世界中，PWA 将从服务器重新加载并重新注册一个新的 Service Worker，重新缓存文件。这对于PWA来说是很大的一笔负担，即使是用户安装了一个 web 应用程序到桌面，在几个星期后，用户仍然不能在离线的情况下访问内容，而只能看到全屏的 “there is no internet” 信息。

此外，苹果此举也限制了Service Workers上的其他API，例如 Background Sync 和 Web Push，它们都依赖于 Service Workers 的持续注册态。

没有事件或其他方式来得知你的 Service Workers 已被销毁，缓存存储已被删除。

此外，与此同时，苹果正式弃用了AppCache。它仍在使用控制台警告。 唯一的问题是，AppCache 的生命周期将作为其他浏览器上的高速缓存存储，这不符合苹果的缓存理念。 所以，AppCache已被弃用，并被替换为不能提供相同行为的其他东西。

好的一方面是，至少我们有了一个 Service Workers 的定义，以及在缓存存储中存储响应的可用空间。 每个分区高达 50 MiB。MiB？ 这是一个[mebibyte](https://en.wikipedia.org/wiki/Mebibyte)，50MiB 是 52.5Mb 左右。 Partition ？ 分区是 Service Workers 的一个新概念，它与iframe有关。 如果你的Web应用程序中没有跨域的iframe，则只需一个分区（你的域和端口）即可。 出于隐私原因，Apple进一步制定了 Service Workers 规范，当一个Service Workers 从跨域 iframe 注册时，会创建新的分区。 Service Workers 将仅负责来自同一分区内的客户端的请求。

我不认为 Service Workers 应该是什么的定义被改变了。 Safari 团队和 Chrome 团队之间曾经有过几次讨论（我在两年前发表[这篇文章](https://medium.com/@firt/service-workers-replacing-appcache-a-sledgehammer-to-crack-a-nut-5db6f473cc9b)的时候在 Twitter 上参与了其中的一些讨论），我想这只是同一个讨论的另一个篇章。 苹果公司并没有参与关于 AppCache 需要替代的讨论，在这里我们得到了结果：他们采用了不同的实现。

### PWAs: 克隆的攻击

Service Workers API 可以在 Safari，Web view（即Chrome，Firefox和Facebook应用内浏览器）上的HTTPS，主屏幕（Web.app）和Safari View Controller中添加的应用程序（如点击 在iOS上的Twitter上的链接）中使用。

听起来不错，对吧？ 那么，这里有一个重要的问题：在Safari上，在每个应用程序的Web view和主屏幕Web应用程序中，引擎不共享服务 Service Workers 和缓存存储，这意味着用户对于同一设备上的PWA可能会有所有文件的多个副本 。

你可能会想：嘿，Max，Android的不同浏览器（Chrome，Firefox，Opera，Samsung Internet，UC Web）也是如此。 那么，你是对的，但有一个不同的用例。 在Android上，操作系统的 Web views 不支持 Service Workers，并且主屏幕上的PWA共享拥有该图标的浏览器的相同 Service Workers 和缓存。 另外，在Android中，同一个设备的不同浏览器上加载相同的网络应用似乎并不是典型的用例。

现在，假设你是iOS用户，并且经常使用PWA，例如Twitter Lite。 当你明确地想要使用它时，你可以打开你的（伪）浏览器，例如iOS上的Chrome或Firefox。 你会得到一个应用程序的副本。 假设你也将其添加到主屏幕中; 应用程序的第二个副本在那里。 因为在iOS上，如果你收到推特上的邮件链接，则无法更改默认浏览器，而必须用Safari打开，这样应用程序的第三个副本将“安装”到你的设备中。就这些吗？ 还没。 如果你使用Facebook或某些报纸应用程序以及其他为您提供应用程序内浏览体验的应用程序，当你单击推特或Twitter帐户的链接时，将拥有另一份副本！ 值得庆幸的是，Safari View Controller似乎与Safari共享Service Workers和缓存。

![](http://onvaoy58z.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-02-09%20%E4%B8%8B%E5%8D%885.48.02.png)

上图是同一PWA在同一设备上的三个不同的 Service Workers（可以通过id区分它们），从左到右分别是：Safari, Chrome 和 Facebook

因此，一个iOS用户可能会在一个设备中得到四个或更多相同PWA的副本（我正在讨论 Service Workers 以及由它缓存的文件，而不是图标）。

## Web App Manifest 支持

当你的HTML链接有一个manifest时，Safari会使用它来代替旧的非标准的apple-mobile- * meta标签。 这很棒！ 但是，就beta 1而言，与Android相比，它还有一些你可能意想不到的行为。

让我们开始谈论哪些属性被忽略（但这是beta 1，我不知道在未来哪些属性会被支持，哪些属性不会）：

+ App name: icon只能使用短名字
+ 主题和背景颜色：没有可着色的状态栏以及闪屏
+ 图标：我昨天看到几位PWA的作者很高兴他们的解决方案在他们的案例安装后正常工作，这不完全正确。 图标来自大多数PWA设置的<link rel =“apple-touch-icon”>，但不是来自Web App Manifest。 我想苹果公司将在未来的beta版本中修复这个问题，我希望图标的尺寸是120x120和180x180。
+ 屏幕方向：无法锁定
+ 展示：全屏：它可以独立工作（您仍然可以使用黑色半透明的状态栏来获得全屏，尽管现在已经被标记为不赞成）
+ 展示：minimal-ui，它作为浏览器的标准快捷方式。


另一方面，我很惊讶，start_url是可用的，这是和以前的添加到主屏幕的巨大变化。 现在使用 pushState 的单页面应用程序在 iOS 上添加到主屏幕时不需要奇怪的技巧。 但是，请记住 manifest 中的 URL 是可见的。

此外，显示模式的CSS媒体查询正常工作。

## 独立模式下的范围和链接

Manifest的作用域在使用<a>元素创建链接时有所不同。 链接应该在PWA shell还是在浏览器中打开？

Android 浏览器通常会打开在 PWA 上下文范围内的URL，并在浏览器或自定义选项卡中打开其他链接。 如果您没有指定Web App Manifest的范围，则Android通常将manifest 的文件夹作为范围，这和我们预期的行为一致。

如果没有指定，Safari将不会定义默认范围，如果是这种情况，那么PWA中的每个链接都将在应用程序的 iOS 窗口中打开。 这会有什么问题？ iOS上没有后退按钮，也没有后退手势，所以用户最终可能会被困在你所链接的外部网站上。 如果你指定范围，一切都按照预期的方式运行，类似于Android，范围内的目标将在Safari中打开  —— 在你的PWA中有一个后退按钮（状态栏中的小按钮）。

## iOS每次都会重新加载PWA

不幸的是，iOS上的主屏幕Web应用程序的一个bug（或者是个feature？）仍然存在。 每当你离开PWA，你将会失去上下文，当用户返回时，PWA将从头开始加载。

考虑到性能，电池使用以及用户期望，这一现象存在更多的问题。 如果你链接到外部网站，iOS上的后退按钮将始终重新加载PWA，这需要花费时间，并且可能不是用户所期望的（你可以用本地存储来伪装，但是这并不理想）。

另外，对于使用双因素身份验证的应用程序（例如Twitter）来说，这是一个巨大的问题。 如果您需要转到其他应用程序以获取令牌或打开短信或电子邮件，则将退出PWA。 当您返回粘贴代码时，您已经脱离了上下文，您需要再次启动登录过程，从而失去该代码的有效性。 这就发生在我的Twitter上！ 这意味着，iOS上的Twitter PWA对我来说是完全无法使用的。

## 欠缺的能力

不幸的是，不支持Web App Banner 或 Manifest 规范事件，比如 appinstalled，所以你需要找到其他的方式来跟踪它（你可能想要使用start_url hash和navigator.standalone来做跟踪）

正如预期的那样，即使推送通知在今天处于死刑的边缘，也没有Web Push Notification 支持。 此外，没有后台同步（Background Sync）支持或Web Share API。 后者十分遗憾，因为从iOS的角度来看，使用原生的 SocialKit 框架应该很简单。

## UX问题和错误

正如我在之前的文章中所说的，iOS有一些差异，比如：

+ 在iOS上，没有物理或逻辑后退按钮或手势。 必须始终在UI内提供它。 这意味着你无法将OAuth登录机制作为顶级页面（无法返回）。 在这里你有一个Twitter Lite的PWA的例子，我没有办法返回或取消操作。 即使我输入了我的电子邮件，我也没有办法再次进入登录页面。

  ![](http://onvaoy58z.bkt.clouddn.com/1_xhGXkMA6wqAG0VX0qRiwzQ.png)

+ 在iOS上，您不能使用透明图标，大多数PWA在Android上都是用圆形图标，在iOS会把相应的透明部分变为黑色，这似乎不是一个好主意。

  ![](http://onvaoy58z.bkt.clouddn.com/1_oFUexHzy3x6Rn7ORMRnxBA.png)

+ iOS上可能存在了五年的状态栏错误仍然存在。 如果你什么都没有做，用户的状态栏会消失，没有时钟，没有电池或者wifi图标。 解决办法仍然是使用status-bar meta 标签，该标签现在接受白色，黑色和黑色半透明作为底色，提供更合适的全屏体验（注意适配 iPhone X）。 由于某种未知的原因，黑色和黑色半透明现在被标记为废弃，将来会被删除。 我认为 manifest 上的主题颜色将优先，但为什么会采用白色而不是黑色？

  ![](http://onvaoy58z.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-02-09%20%E4%B8%8B%E5%8D%885.46.31.png)

## 我们仍有时间作出改变

苹果仍然有空间做一些改变，让我们的生活更轻松。 与此同时，我们也有时间检查我们的PWA，例如

- 图标：添加符合iOS大小的图标，没有透明边缘
- 如果有任何<a>标签，则需要在界面中加上后退按钮
- 在 Web App Manifest 中规定范围
- 如果你要求用户离开应用并返回，鉴于重新加载的逻辑，该怎么处理
- 如何促进用户安装你的应用程序（更新支持iOS的提示）
- 如何跟踪PWA安装

有其他的想法？ 记得在 [Twitter](http://twitter.com/firt) 上关注我，我将经常测试新的 beta 版本并更新信息。

