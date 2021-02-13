---
title: 你应该知道的 7 个新的 Chrome API
date: 2020-01-20 10:50:44
tags: 
  - 翻译
---

原文作者：[Chidume Nnamdi 🔥💻🎵🎮](https://blog.bitsrc.io/7-new-chrome-apis-you-should-know-cf2dcb9f42dc)

译者：Sangle

---

> 这7 个 Chrome 的新功能，模糊了 Web 应用和原生应用程序之间的界限 

我们喜欢原生应用程序，因为它们似乎提供了更好的用户体验：更快，更可靠。 不过，我们并不想把每个 web 应用程序都做成原生程序。 随着网络的发展，我们看到 web 应用与原生应用程序之间的界限越来越模糊 —— 这是有充分理由的 😃

现在，我将给您展示最新的令人兴奋的技术，这些技术正在我们的浏览器中使用。 它们中的大多数早已内置在浏览器中，有一些人已经足够快地进行了简单试验（比您想象的要早），我们将在浏览器中看到这些 API 带来的全新功能，尤其是在 Chrome 浏览器中。

## 1. Web Bundles

![Web Bundles](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/202003091.png)

此 API 可以将整个网站打包为一个文件。 不仅如此，您还可以在任何介质（蓝牙或Wi-fi Direct）上共享这个文件，并且该文件可以脱机运行。

捆绑的物品是一个`.wbn`扩展文件。 所有HTML页面，CSS，图像，视频等都打包在 `.wbn` 文件中。

<!-- more -->

想象一下，您正在手机上使用 Web绘图应用程序，然后您注意到您的流量将很快用尽。 保留剩余的数据后，您可以使用 Web Bundle API 将 Web绘图应用程序 打包到手机上。

接下来，您知道数据已经用完，不用担心，您可以在应用程序中打开下载的 Web绘图应用程序包，然后继续进行工作。

您和朋友一起爬山，在山顶上没有网络信号，但该应用程序仍然正常运行。 您的朋友看到了它，表示很喜欢，您将 `.wbn` 文件发送给他。 他打开手机上的 `.wbn` 文件，开始绘制，没有问题。



更多信息：<https://web.dev/web-bundles/>



## 2. 定期后台同步

![定期后台同步](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/202003092.jpeg)

定期后台同步是后台同步的扩展，它允许网站注册的 service worker 通过网络连接定期运行任务。

到目前为止，它非常简单，而且只能在原生应用程序（Android的AsyncTask和Thread等）中完成。

这对于提前刷新用户供稿的内容非常有用，或者在该PWA新闻应用中，我们可以在用户阅读新闻的同时定期更新新闻，而不会影响用户体验。 不需要刷新按钮（就像Twitter的下拉刷新一样 :)

该 API 在低网络带宽，较差的网络接收或故障网络的领域中非常出色。

此 API 在Chrome 77中可用。

更多信息：<https://web.dev/periodic-background-sync/>



## 3. Web Share API

该 API 带来了类似原生的共享功能，我们过去只能在原生应用程序中看到这些功能。 借助此API，我们可以共享链接，文本和文件，就像我们在原生应用程序中所做的一样。

我们还可以使用此 API 注册我们的Web应用程序（PWA）以接收共享数据。

Chrome 68 提供了此API。

查看我的Web Share API示例：<https://web-share-api.github.io/>



## 4. Web Contact Picker

![Web Contact Picker](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/202003093.jpeg)

使用 Web Contact API，允许用户从其联系人列表中选择条目，并与网站共享所选条目的有限详细信息。

在这里，我们再次看到了向 Web 应用程序引入的纯原生功能（在Android中，您可以使用Intent.ACTION_PICK访问联系人列表）

![Web Contact Picker](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/202003094.png)

使用此 API，我们可以拥有一个 Web 应用程序，该应用程序使用户可以从其联系人列表中选择一个朋友以通过网络进行呼叫。 使用联系人列表选择要发送电子邮件的朋友，选择要 Skype 的朋友或与之一起使用任何社交网络的Web 应用程序。

此 API 在 Chrome 77中可用。

更多信息：<https://web.dev/contact-picker/>



## 5. Wake Lock API

唤醒！唤醒！

我们看到许多本机应用程序使用唤醒锁定功能来防止我们的设备在使用应用程序后经过指定的空闲时间后进入睡眠状态。

游戏就是很好的例子，它们可以防止手机在游戏开启时进入休眠状态。 您在玩 Temple Run，并且暂停了。 您的手机将闲置在床上，Temple Run 将使用唤醒锁来防止手机进入睡眠状态，直到您返回手机为止，手机都将保持苏醒状态。

似乎只有原生应用程序可以做到这一点。 现在，此唤醒锁定功能即将出现在我们的浏览器中，我们将能够在我们的 Web 应用程序中使用它。

借助 Wake Lock API，我们可以防止设备在空闲时间进入休眠和锁定状态。

唤醒锁 API 的用例有很多。

查看更多信息：[Stay awake with the Wake Lock API](<https://web.dev/wakelock/>)



## 6. `getInstalledRelatedApps()`

使用此API，我们可以检查设备上是否已安装对应 web 应用程序的的原生应用。

查看更多信息：[Is your native app installed? getInstalledRelatedApps() will tell you!](<https://web.dev/get-installed-related-apps/>)



## 7. Shape Detection API

![Shape Detection API](https://sangle-1256136582.cos.ap-guangzhou.myqcloud.com/202003095.jpg)

使用此 API，您可以从浏览器中检测形状。

该 API 当前支持检测以下功能：

**人脸**：通过 FaceDetector 接口，此 API 可以检测图像中的人脸。想象一下拍一张照片，您的浏览器会告诉您图像中谁是谁。像 FaceBook 这样的大多数社交网络都这样做，您上传一张图像，它通过面部检测告诉您这是您的朋友 Frida，这是您的朋友 John。

通过这种面部检测，我们可以突出显示个人资料图片中的面部并对其进行缩放。

我们也可以使用此方法在图片上添加叠加层。像 Snapchat 一样，上面的覆盖层如 Thor 的胡须，太阳镜，表情符号。

**条形码**：通过 BarcodeDetector 界面，我们可以从 Web 应用程序中读取条形码。

如今，条形码主要用于超市和杂货店。价格存储在商品中的条形码中，杂货店招标使用条形码读取器读取价格并标记每个购买的商品并告诉您价格。

现在，使用具有杂货店女售货员功能的浏览器，您可以通过手机四处走动，直接从Web应用程序查看价格！

条形码不仅在商店中使用，在机场，书籍，在线支付中也会使用。

**文本**：我们可以使用 Shape Detection API 中的 TextDetector 接口检测文本。

您会看到，这带来了很多可能性。通过这种文本检测，我们可以使用我们的Web应用程序来检测文本并将其翻译为我们理解的语言。

查看更多信息：[The Shape Detection API: a picture is worth a thousand words, faces, and barcodes](https://web.dev/shape-detection/)



## 总结

这是我列出的最令人兴奋的功能。 在下面发表评论，让我知道你的想法😃

干杯! 🍺