---
title: "【构建Android缓存模块】（一）吐槽与原理分析"
date: 2014-06-04 22:32:52 +0800
comments: true
categories: [Android]
desc: 在我翻译的 [Google 官方系列教程][1]中，Bitmap 系列由浅入深地介绍了如何正确的解码 Bitmap ，异步线程操作以及使用 Fragments 重用等技术，并且在最后给出了非常强大的独家秘笈：BitmapFun ，让猿媛们得以一窥究竟 Google 的攻城师们是如何高屋建瓴地秒杀 OOM 的。
---

## 摘要

在我翻译的 [Google 官方系列教程][1]中，Bitmap 系列由浅入深地介绍了如何正确的解码 Bitmap ，异步线程操作以及使用 Fragments 重用等技术，并且在最后给出了非常强大的独家秘笈：``BitmapFun`` ，让猿媛们得以一窥究竟 Google 的攻城师们是如何高屋建瓴地秒杀 **OOM** 的。

## 前言

在下载到 [BitmapFun.rar][2] 这个神圣的压缩包以后，我是双手颤抖，似乎是打开上古秘藏一般，心情激动导致久久不能自已。我还记得那天上海下着小雨，我当时霍然起身，伫立在 23 楼的窗台，仰着头向江水对岸的东方明珠望去，似乎这样我郁积已久的眼泪就不能掉下来。说到这里，Ryan 又暗自抹了一把眼泪。短暂地忘记了过去的黑暗时光，那一个漫长的被 **OOM** 的淫威所折磨的盛夏。。。

最后在 Boss诧异的目光中，我回到办公桌，按捺着内心汹涌的情绪波动，然后小心翼翼的打开 BitmapFun.rar 。当那些在洪荒时代就活跃在Android平台的大师们书写的篇章呈现在我眼前时，我的表情与阿宝从师父手里得到 **Dragon Scroll** 时一般，永久的定格在了极度天真的期待与眼角一抽一抽的状态。

那些泛黄的代码在我看去，通篇只有一句话：``老子看不懂！``

## 自力更生，构建自己的缓存模块

Google 的这个 demo 堪称详尽，考虑极其周详，自然是极好的。但是当原理被层层的“特殊情况”包装起来，原本简单的例子变得异常复杂，几个类之间的关系错综复杂，堪比吸血鬼日记几个帅哥美女之间的关系。要理解清楚每一句代码的含义，你一定要有理解 Matt 那人老珠黄的老娘和他和失落的好朋友 Taylor 搞在一起的觉悟。

好了，吐槽一下就收，千万不要怀疑 Google ，人家已经仁至义尽了。 BitmapFun 中在下载后将 Bitmap 缓存起来，缓存做了两份：``LruCache`` 和 ``DiskLruCache`` ，分别是内存缓存和硬盘缓存。此外两个至关重要的类是：

```java
BitmapWorkerTask(ImageView imageView)
    
AsyncDrawable extends BitmapDrawable
    AsyncDrawable(Resources res, Bitmap bitmap,BitmapWorkerTask bitmapWorkerTask)    
```

``BitmapWorkerTask`` 持有一个 ``WeakReference<ImageView> imageViewReference`` ，弱引用 ``ImageView`` ，用作异步处理加载图片的任务。
``AsyncDrawable`` 巧妙的引用持有弱引用 ``WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference`` ，是 ``BitmapDrawable`` 的子类，这样就可以 ``setImageBitmap(AsyncDrawable)``。

关系：``AsyncDrawable`` 中弱引用 ``BitmapWorkerTask`` 。其实是图片引用 ``ImageView`` 的关系，而``ImageView.getDrawable`` 又可以获得图片。这种高妙的思想不是正值得我们学习么？   

当然，这节课并不是讲解官方Demo的，在讲解它之前，我们先来学习一个更加简单的缓存实现方案，使用最简单的方式快速构建自己应用的缓存模块，有效避免 OOM 异常。它的难度非常小也很方便理解，可以在这个缓存实现的基础上，我们再去理解更加高妙的 BitmapFun 的缓存实现方案。

## 如何解决OOM

Bitmap 之所以容易引起OOM异常，原因已经在 Bitmap 系列教程中说的明明白白。但是我们至少清楚一点：一个手机屏幕再大，合理尺寸的 Bitmap 也不至于耗空所有内存，那要怎么做才能避免 OOM 呢？

- 加载合理尺寸的 Bitmap 
- 避免反复解码、重复加载 Bitmap
- 控制 Bitmap 的生命周期，合理回收

此外网上也有不少歪门邪道，我个人认为是不可取的，使用这些简单粗暴的方法，后期会为你带来更大的麻烦：

- 减损图片质量（使用过高的 inSampleSize 值）
- 使用 decodeStream （绕过 Java 层，直接调用 JNI ）
- 强制增加heap size
- 其他

控制Bitmap的生命周期才是正解， BitmapFun 使用的 LruCache 是将它将最近被引用到的对象存储在一个强引用的 LinkedHashMap 中，并且在缓存超过了指定大小之后将最近不常使用的对象释放掉。

Memory Cache 的 Size 是受限的，因此加入 DiskLruCache ，虽然在访问速度上逊于 Memory Cache ，但是速度也是相当可观的。

借鉴 Google 的做法，我也将缓存做了两份，一份是 Memory Cache ，使用弱引用的 WeakHashMap 来控制 Bitmap 的生命周期，后面会有详细解释。另一份严格来说不能算是缓存，直接将文件存储在 SDCard 上，避免重复下载。

## 佛说引用，既非引用，是名引用。

关于引用，或许对于小菜鸟们不是很好理解（我碰到过太多 Java 都没学好来做 Android 的，基础很重要！）。我使用金刚经的著名三段论来解释它：``佛说XX，既非XX，是名XX。``

这句话什么意思呢？比如佛说大米，既可以说它不是大米，只是名字叫做大米罢了。不会因为你为它改名叫做大麦而改变它的本质，你叫它做水，吃到嘴里的还是原来的味道。

关于引用，跟这个有着非常相似的共性。引用就相当于实际对象的名字，比如下面的例子：

```java
Person p1 = new Person();
Person p2 = null;
p2 = p1;
p1 = null;
```
``new Person()``这个对象的名是 p1 ，而后你将名字改成了 p2 ，对象还是那个对象，不会因为你将 p1 的大名盖在 null 的头上而改变它的本质。以上的 p1 和 p2 都是引用，它们都不过是名。

> 在了解到引用的含义后，虚拟机会告诉你，被引用的对象处于可获得( reachable )状态，它是你的好管家，既然你要用它，它就不会回收它。（你想想如果你正在吃一只烤鸭，人家突然一把抢了过去扔垃圾桶了你什么感觉。）

> 如果在上面的那段程序后面加上 ``p2 = null``，Person 这个对象就没有任何引用指向它了，垃圾回收器会在不确定的时间进行回收。（你都把东西扔了，总不能不让人家收破烂吧？）

> 如果你想继续持有这个对象的引用，希望可以继续访问，但是也允许垃圾回收器进行回收，该怎么办呢？（你想减肥，告诉你的好朋友说，如果察觉到你太胖了，就将你嘴里的烤鸭抢去扔了。如果你很饿，身材也不错，你要继续吃。）

这个时候，我们需要借助 Java 提供的软/弱/虚引用。我们平时使用的如 p1 和 p2 这样的叫做强引用(Strong Reference)。要使垃圾回收器能在内存不够的时候，主动抢下你嘴里的烤鸭，进行回收，需要使用这些：

- 软引用：SoftReference
- 弱引用：WeakReference
- 虚引用：PhantomReference

它们按照由强到弱的引用关系排列，虚引用相当于几乎没有引用。文艺青年常说的若即若离用来形容它再恰当不过了。

关于这三个引用的具体学习，详见我提供的参考资料。这里只是向你解释为什么使用弱引用可以起到防止 Bitmap 过多而导致内存紧张的作用。

在这里，由于我需要使用 Bitmap 和名字的 key-value 对应关系，我使用 Java 提供的 ``WeakHashMap(String key, Bitmap value)`` ，顾名思义，它用来保存 WeakReference ，并且确保每个 key 只对应一个值，在内存不够的时候，垃圾回收器会进行回收。当 key 值索引不到 Bitmap ，再进行其他的操作。

## 原理示意图

我将原理画成图，以便大家的理解。主体有三个，分别是 UI ，缓存模块和数据源。它们之间的关系如下：

![原理示意图][3]

① UI：请求数据，使用唯一的 Key 值索引 Memory Cache 中的 Bitmap 。

② 内存缓存：缓存搜索，如果能找到 Key 值对应的 Bitmap ，则返回数据。否则执行第三步。

③ 硬盘存储：使用唯一 Key 值对应的文件名，检索 SDCard 上的文件。

④ 如果有对应文件，使用 BitmapFactory.decode* 方法，解码 Bitmap 并返回数据，同时将数据写入缓存。如果没有对应文件，执行第五步。

⑤ 下载图片：启动异步线程，从数据源下载数据(Web)。

⑥ 若下载成功，将数据同时写入硬盘和缓存，并将 Bitmap 显示在 UI 中。

总结：这节课除了吐槽，主要的还是原理分析。如果你有更好的缓存方案，欢迎提出。下节课将讲解具体的 Memory Cache 和 FileCache 如何实现。

## 参考资料

【1】[Thinking In Java 4th Chapter 17.12 Hoding References.pdf][4]

【2】[李刚：突破程序员基本功的16课之Java的内存回收.pdf][4]  

[1]: http://my.oschina.net/ryanhoo/blog?catalog=260281
[2]: http://vdisk.weibo.com/s/hNgFB
[3]: /images/blog/android/130726_4p9B_245415.png
[4]: http://vdisk.weibo.com/s/jtqjr
[5]: http://vdisk.weibo.com/s/jtqik