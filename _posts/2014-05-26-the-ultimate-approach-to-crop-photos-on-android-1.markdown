---
title: Android 大图片裁剪终极解决方案（上：原理分析）
date: 2014-05-26 17:37:42 +0800
comments: true
categories: [Android]
desc: 约几个月前，我正为公司的 APP 在 Android 手机上实现拍照截图而烦恼不已。上网搜索，确实有不少的例子，大多都是抄来抄去，而且水平多半处于 demo 的样子，可以用来讲解知识点，但是一碰到实际项目，就漏洞百出。当时我用大众化的解决方案，暂时性的做了一个拍照截图的功能，似乎看起来很不错。可是问题随之而来，我用的是小米手机，在别的手机上都运行正常，在小米这里却总是碰钉子。虽然我是个理性的米粉，但是也暗地里把小米的工程师问候了个遍。真是惭愧！
---

约几个月前，我正为公司的 APP 在 Android 手机上实现拍照截图而烦恼不已。

上网搜索，确实有不少的例子，大多都是抄来抄去，而且水平多半处于demo的样子，可以用来讲解知识点，但是一碰到实际项目，就漏洞百出。

当时我用大众化的解决方案，暂时性的做了一个拍照截图的功能，似乎看起来很不错。可是问题随之而来，我用的是小米手机，在别的手机上都运行正常，在小米这里却总是碰钉子。虽然我是个理性的米粉，但是也暗地里把小米的工程师问候了个遍。真是惭愧！

翻文档也找不出个答案来，我一直对 **com.android.camera.action.CROP** 持有大大的疑问，它是从哪里来，它能干什么，它接收处理什么类型的数据？Google 对此却讳莫如深，在官方文档中只有Intent中有只言片语言及，却不甚详尽。

随着项目的驱动，我不能抱着不了解原理就不往前走的心态，唯一要做的，是解决问题。最后在德问上找到一条解决方案，说是哪怕是大米也没问题。当时乐呵呵将代码改了改，确实在所有的手机上跑起来了，一时如释重负，对这个的疑问也抛诸脑后了。

直到月前，BOSS要求将拍照上传到服务器的图片分辨率加倍。OK，加倍简单，增加 ``outputX`` 以及 ``outputY`` 不就得了？

```java
intent.putExtra("outputX", outputX);
intent.putExtra("outputY", outputY);
```

这一增加，吓了我一跳。BOSS 的手机拍到的照片几乎就是个缩略图，但是被我问候了全体工程师的小米在这个时候就体现出国产神机的范儿了，小米上的尺寸一切正常。这个为什么呢？我大致了解原因，却不知道如何解决。

在 Android 中，Intent 触发 Camera 程序，拍好照片后，将会返回数据，但是考虑到内存问题，Camera 不会将全尺寸的图像返回给调用的 Activity，一般情况下，有可能返回的是缩略图，比如**120*160px**。

这是为什么呢？这不是一个 Bug，而是经过精心设计的，却对开发者不透明。

以我的小米手机为例，摄像头800W像素，根据我目前设置拍出来的图片尺寸为 **3200*2400px**。有人说，那就返回呗，大不了耗 1-2M 的内存，不错，这个尺寸的图片确实只有1.8M左右的大小。但是你想不到的是，这个尺寸对应的 Bitmap 会耗光你应用程序的所有内存。Android出于安全性考虑，只会给你一个寒碜的缩略图。

在 Android2.3 中，默认的Bitmap为32位，类型是``ARGB_8888``，也就意味着一个像素点占用4个字节的内存。我们来做一个简单的计算题：

`` 3200*2400*4 bytes =   30M ``

如此惊人的数字！哪怕你愿意为一张生命周期超不过 10s 的位图愿意耗费这么巨大的内存，Android 也不会答应的。

> Mobile devices typically have constrained system resources. 
> Android devices can have as little as 16MB of memory available to a single application.

这是 Android Doc 的原文，虽然不同手机系统的厂商可能围绕16M这个数字有微微的上调，但是这 30M，一般的手机还真挥霍不起。也只有小米这种牛机，内存堪比个人 PC，本着土财主般挥金如土的霸气才能做到。

OK，说了这么多，无非是吐吐苦水，爆爆个人经历而已，实际的解决方案在哪里呢？

我也是 Google 到的，话说一般百度不了的问题，那就``Google``或者直接``StackOverFlow``，只不过得看英文罢了。

最后翻来覆去，我在国外的一个 Android 团队的博客中找到了相应的方案，印证了我的猜想同时也给出了实际的代码。

我将这篇文章翻译成了中文，作为本博客的基础，建议详细看看。

[【译】如何使用Android MediaStore裁剪大图片][1]

这篇博客了不起的地方在于解决了 Android 对返回图片的大小限制，并且详细解释了裁剪图片的 ``Intent`` 附加数据的具体含义。我只是站在巨人的肩膀上，改善方案，适应更广泛需求而已。

拿图说事儿：

![Intent Options][2]

``Intent("com.android.camera.action.CROP")`` 对应的所有可选数据都一目了然。在了解上面个个选项的含义之后，我们将目光着眼于三个极为重要的选项：

- data
- MediaStore.EXTRA_OUTPUT
- return-data

``data 和 MediaStore.EXTRA_OUTPUT`` 都是可选的传入数据选项，你可以选择设置 data 为 Bitmap，或者将相应的数据与URI关联起来，你也可以选择是否返回数据（return-data: true）。

为什么还有不用返回数据的选项？如果对 URI 足够了解的话，应该知道 URI 与 File 相似，你所有的操作如裁剪将数据都保存在了 URI 中，你已经持有了相应的 URI，也就无需多此一举，再返回 Bitmap 了。

前面已经说到，可以设置 data 为 Bitmap，但是这种操作的限制在于，你的 Bitmap 不能太大。因此，我们前进的思路似乎明确了：截大图用 URI，小图用 Bitmap。

我将这个思路整理成一张图片：

![idea][3]

这篇主要让大家了解需求的来源，以及如何去思考分析并解决问题。下一篇博客将介绍具体的操作。

[1]: http://my.oschina.net/ryanhoo/blog/86843
[2]: /images/blog/android/144805_wCcI_245415.png
[3]: /images/blog/android/151831_7gRC_245415.png
