---
title: 【疑难杂症】Android WebView 无法打开天猫页面
date: 2014-09-17 11:14:25 +0800
comments: true
categories: [Android, WebView, Tricky Bugs]
desc: Android WebView 突然无法打开天猫的详情页，一直停留在加载状态。而在此之前，应用里是完全可以正常访问的，简单设置一行代码 webView.getSettings().setDomStorageEnabled(true) 即可解决，但背后的原因又是什么呢？
---

> 欢迎转载，但请务必注明出处！
>
> [http://ryanhoo.github.io/blog/2014/09/17/android-webview-setdomstorageenabled][9]

Android WebView 突然无法打开天猫的详情页，一直停留在加载状态。而在此之前，应用里是完全可以正常访问的，通过搜索，找到解决方法，简单设置一行代码 
``webView.getSettings().setDomStorageEnabled(true)`` 即可解决问题，但背后的原因又是什么呢？

![Android WebView Can't Load TMall Pages][1]

我们不能只是做解决问题的程序员，还要做好奇的程序员，跟我来一探究竟吧。

## 定位问题

本着**发现问题 -> 分析问题 -> 解决问题**的步骤，接下来我们要定位问题的来源。

## Chrome Inspcet Device

我首先想到的是，Android 4.4 以后的 WebView 是基于 chromium 内核的，这带来了一个福利是，我们可以在 chrome 中 inspect device，像普通网页一样，进行 debug，精准、快速定位问题来源。正好我手头有 Nexus 5 刷的是 **Android L Preview** 还有 Mi3 是最新的 **MIUI V6**，打开 Chrome，``Tools`` -> ``Inspect Devices``，就可以看到原生应用里的 WebView 页面了。

![Chrome Inspect Devices][2]

点击 ``inspect`` 即可进入 **Developer Tools**，一切尽在掌控，可以清楚看到 console 输出的错误信息。

![Chrome Developer Tools][3]

## Android Logcat

实际上，网页加载中 js 输出的错误信息也完整的打印在了 Logcat 中。

```java
09-17 11:28:20.265  25993-25993/ I/chromium﹕ [INFO:CONSOLE(2)] "TypeError: Cannot read property 'hotelLon' of null
  at BaseDerived.g.extend.loadItemData (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??pages/tuan/detail.js:2:8436)
  at BaseDerived.g.extend.initializer (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??pages/tuan/detail.js:2:340)
  at BaseDerived.initializer (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:24184)
  at BaseDerived.Base (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:24486)
  at BaseDerived.S.extend.callSuper (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:26305)
  at BaseDerived (eval at extend (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:27954), <anonymous>:1:51)
  at BaseDerived.S.extend.callSuper (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:26305)
  at new BaseDerived (eval at extend (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:27954), <anonymous>:1:51)
  at Object.<anonymous> (http://h5.m.taobao.com/trip/hotel/tuan/detail.html?itemId=40987076007:426:22)
  at o (http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918:3:4799)", source: http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918 (2)
09-17 11:28:20.265  25993-25993/ I/chromium﹕ [INFO:CONSOLE(2)] "Uncaught TypeError: Cannot read property 'hotelLon' of null", source: http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??libs/seed.js,config.js,widgets/bridge/index.js,widgets/mtop/index.js?v=1203354280_108918 (2)
```

## 分析问题

### 追根溯源

以上两种方式找到的错误信息（相同的）列出了错误栈信息，第一条错误是发生在 ``http://g.tbcdn.cn/trip/h5-hotel/0.3.52/??pages/tuan/detail.js`` 的第 2 行第 8436 列。在浏览器中打开这个 js 文件，显而易见它是被压缩过的，复制在 IDE 如 Sublime Text 中格式化，根据错误信息 ``"TypeError: Cannot read property 'hotelLon' of null`` 搜索 **hotelLon**，可以找到相关代码。

```javascript
loadItemData: function() {
	var a = this,
		c = !1;
	a.loading.show(), a.curLongitude = a.getRequestParam("longitude") 
		|| a.getRequestParam("lng") 
		|| localStorage.hotelLon 
		|| "", a.curLatitude = a.getRequestParam("latitude") 
		|| a.getRequestParam("lat") 
		|| localStorage.hotelLat 
		|| "", a.fetchItemData(function(d) {
		return a.loading.hide(), a.detailData = c ? MockData.data : d.data, $.isEmptyObject(d.data) && !c ? d.ret && d.ret[0] ? void a.showError(".J_detail", d.ret[0].split("::")[1].split("##")[0] + o) : void a.showError(".J_detail", "\u4eb2\uff0c\u8be5\u9152\u5e97\u6682\u65e0\u56e2\u8d2d\u4fe1\u606f\u3002" + o) : "1" == a.detailData.isPreItem ? void(window.location.href = b("http://h5.m.taobao.com/awp/core/detail.htm?id=${itemId}&ttid=${ttid}", {
			itemId: a.getRequestParam("itemid"),
			ttid: a.ttid
		})) : (a.detailData = a.dirtyDataProcess(a.detailData), a.render($(".J_detail"), a.detailData), a.renderTitle(a.detailData), a.countdownInit(), a.submitButtonStatusInit(), n.init(function() {}), a.sliderInit(), a.detailData.propertiesMap && a.skuModule.init($(".J_sku"), a.detailData), a.priceModule.init($(".J_discount")), a.loadRateInfo(a.detailData.hotelList[0].hid, a.detailData.hotelList[0].shid), a.bindSliderClick(), a.bindServiceClick(), a.bindRateClick(), a.bindSubHallClick(), a.bindDescClick(), a.bindMapClick(), a.bindSkuModuleEvent(), a.bindRecommendItemClick(), a.bindFavoriteClick(), a.bindShareClick(), a.bindSubmitEvent(), a.isTripClient() && ($(".J_discount").removeClass("sticky"), $(".J_favorite").hide(), $(".fav-line").hide()), i.locStorage.set("hotel-tuan-detailHotelList", JSON.stringify({
			hotelList: a.detailData.hotelList
		})), $(".call-seller").parent().on("touchstart", function() {
			a.sendPoint("Button", "TuanHotel", "Call")
		}), void $(".wangxin-waptowx").on("touchstart", function() {
			a.sendPoint("Button", "TuanHotel", "Contact")
		}))
	}, function() {
		a.loading.hide(), a.showError(".J_detail", "\u4eb2\uff0c\u7f51\u7edc\u5f00\u5c0f\u5dee\uff0c\u8bfb\u53d6\u6570\u636e\u5931\u8d25\u3002" + o)
	})
},
```

### 代码分析

在上面这段代码的 loadItemData 方法中，第 6 行 ``localStorage.hotelLon``，在这里读取了 localStorage 的 hotelLon 信息，顾名思义，就是酒店的经度，但是提示的错误信息是无法读取 hotelLon。如果对 localStorage 有所了解的话，应该知道即便没有 hotelLon，也不会报错，而是 **undefined**。

![localStorage fetch a non-existed variable][4]

#### localStorage

到这里，需要补充一下 localStorage 的知识。

HTML5 Storage主要有：

- sessionStorage: 会话 (session) 级别的数据存储，会话结束后，相关的数据就会被清除掉。
- localStorage: 用于持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的。

作为 HTML5 标准的一部分，绝大多数的浏览器都是支持 localStorage 的，但是鉴于它的安全特性（任何人都能读取到它，尽管有相应的限制，将敏感数据存储在这里依然不是明智之举），Android 默认是关闭该功能的。

## 解决问题

经过层层分析，问题指向如何开启 localStorage 了。

```java
// Gets whether the DOM Storage APIs are enabled.
webView.getSettings().setDomStorageEnabled(true);
```

## 参考
1. [DOM Storage guide][5]
2. [Dive into HTML5: LOCAL STORAGE][6]
3. [HTML5之Web Storage(本地存储)详解][7]
4. [StakeOverflow: What does “enable DOM storage API” mean?][8]

[1]: /images/blog/android/34cb73b6bb0dfbac04b1f50d6a58c05a.png
[2]: /images/blog/android/d5681ec2448ebb4bfd6e22eb805d7ad6.png
[3]: /images/blog/android/45c5b879756949cb1e70523f5aad0276.png
[4]: /images/blog/android/e6ff9ad286da4dae4c75ea3613980ff5.png
[5]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage
[6]: http://diveintohtml5.info/detect.html
[7]: http://blog.baiwand.com/?post=184
[8]: http://stackoverflow.com/questions/5858760/what-does-enable-dom-storage-api-mean
[9]: http://ryanhoo.github.io/blog/2014/09/17/android-webview-setdomstorageenabled