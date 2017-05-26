---
title: 【译】使用 Fragment 自定义动画
date: 2014-09-30 21:32:45 +0800
comments: true
categories: [Android, Animation, Fragment]
desc: 在过去的几个月里，我一直忙于从无到有的开发一个 Android 应用。这个应用以我公司的名字命名 —— Capitaine Train，可以在 Google Play 商店下载。Capitaine Train 从字面翻译成英文是 Captain Train，这是一个有着 3 年历史的创业公司，它秉持一个简单的理念而生：在欧洲买火车票是一件极为令人痛苦的事情，而我们 —— Capitaine Train 致力于通过简化整体乘车体验来彻底变革人们来往欧洲各国旅行的方式。这一版的 Android 应用意味着我们在这一方向上迈出了重要的一步。
---

> 作者：[Cyril Mottier][author]
>
> 原文：[Custom Animations With Fragments][origin post]

**作者按**：一般来说，我发表的博客通常都是与我日常工作无关的。但是下面这篇文章提及了我在 Capitaine Train 所做的一些工作。因此，我觉得需要特此声明：我在 Capitaine Train 工作，但是我在博客以及其它任何地方（Twitter, Google+ 等）表达的观点，仅限于我个人，与我的雇主无关。

在过去的几个月里，我一直忙于从无到有的开发一个 Android 应用。这个应用以我公司的名字命名 —— Capitaine Train，可以在 Google Play 商店下载。Capitaine Train 从字面翻译成英文是 Captain Train，这是一个有着 3 年历史的创业公司，它秉持一个简单的理念而生：在欧洲买火车票是一件极为令人痛苦的事情，而我们 —— Capitaine Train 致力于通过简化整体乘车体验来彻底变革人们来往欧洲各国旅行的方式。这一版的 Android 应用意味着我们在这一方向上迈出了重要的一步。

试图去变革欧洲的乘车体验绝对不是一件简单的事情。这意味着我们要完成大量的工作：了解各种各样的铁路运营商、研究他们各自的文档以及预订要求、集成他们的价格/时间表、将他们的服务接口绑定到我们的系统，等等等等。从用户的角度来看，这一切都是隐藏的，但却是至关重要的，而这只是冰山一角。确实，一次旅行的需求或者愿望起始于一个简单得不能再简单的搜索请求：从哪里来？到哪里去？什么时候？跟谁？尽管这些问题非常简单，但搜索的步骤在整个预订的环节是至关重要的。毕竟这是一次旅行的起点。在设计这个 Android 应用的过程中，我们始终不忘最初的理念，尽可能简化每一步的操作流程。在这篇文章中，我很乐意跟你们分享我是如何实现这个搜索体验以及我们如何利用动画来提升用户体验。

## 从网页端到移动端

当我到 Capitaine Train 并开始着手 Android 应用的开发工作时，我先浏览了当下所有进行中的项目产品。比如 iOS 应用，当时还没有发布，但是已经快速成型了。另外的，比如我们的 Web 端应用，已经公开面向大众并且在用户中非常受欢迎。我那时的主要工作是构建一个能让用户觉得他们在用最好的 Android 应用订票的产品。而且这个应用必须反应出 Capitaine Train 的本质以及Android 风格的外观。由于当时 Web 应用是唯一发布的，显然我大部分的草稿都是基于它的。这是 [capitainetrain.com](capitainetrain.com) 的搜索表单：

![search form][search form]

[播放 mp4][search form mp4]

两列（搜索表单 + 选项）的设计在网页上表现很完美，但我们很快就在移动端面临一个难题：我们没有足够的空间把表单和选项面板都放在一个屏幕里。因为手机屏幕实在是太小了，别无选择，我们只好回到 master/detail（主从） 模式。此时有两个众所周知且简单可行的选项摆在我们面前：master/detail（主从） 模式和 edition dialogs（对话框编辑） 模式。但是我们对这些模式并不满意，实际上，对话框完全打断了用户操作流程，并且在表单中至少有 4 个地方需要对话框，这将会极为令人生厌。从另一方面来说，如果每编辑一个信息就打开一个全屏的『选项』 Activity，这样高度复杂的屏幕层级关系和应用结构会导致一部分用户的流失。我想这些模式都不够有效也不适合 Capitaine Train 的 Android 应用。

我们当然想复制 Web 网页上简洁明了的搜索方式，最终我们想到一个完美的方案。与其为每一个表单字段打开一个模态视图，我们设法将表单栏和选项面板合并在一个屏幕内。默认情况下，应用显示所有可供选择的搜索表单。点击其中任何一个字段将会切换到『编辑模式』，而此时编辑的字段在顶部可见，表单的其余部分则隐藏起来以显示该字段对应的选项面板。下面的视频显示了完整的搜索流程使用示例：

![search android][search android]

[播放 mp4][search android mp4]

通过我们精心设计的过渡动画，上面的示例看起来表现不错。确实，如果没有这些过渡动画的话，这些功能将会变得无法使用。给应用加入适当的过渡动画是一个丰富用户体验的好办法，它让用户直观的理解到他们每一个行为所带来的结果。如牛顿所说，每一个行动必有一个结果：过渡解释了 UI 切换间的变化。从一个屏幕切换到另一个屏幕时，它能避免给人带来『屏幕堆砌』的印象。它让用户觉得整个应用都是由一个屏幕构成的，里面的 UI 元素通过动画来显示或者隐藏应用的某一部分。换句话说，过渡动画打破了传统疆界并使得应用导航更为自然。

## 过渡动画分解

过渡动画通常很快而且一闪即逝令人难以察觉。为了更好的理解动画细节、创建动画以及逆向工程可以考虑让动画慢下来，结果将会非常有意思。如果你有应用源码的话，显然可以增加动画的时间值。不然的话，你需要录制应用的操作视频然后一帧一帧的看慢动作。幸运的是，Android 给我们提供了一个非常有用的技巧：开发者模式下有一个选项叫做『动画时长调整』(Animator duration scale)，顾名思义，这个选项可以调整系统级别的动画时长。

为了更好的理解当搜索表单与时间编辑模式切换时发生了什么，让我们用上面提到的技巧来一窥究竟。下面的屏幕录制是在普通动画 10 倍（10x）的时长下进行的：

![search android slow][search android slow]

[播放 mp4][search android slow mp4]

通过上面慢放的操作视频，我们可以细致的观察编辑模式的渐变。更具体地说，你可能已经注意到渐变最终被分成了几个同时进行的子动画，它们有着共同的时间属性（duration、interpolator 等等）。

[未完待续...][origin post]

[author]: http://cyrilmottier.com/
[origin post]: http://cyrilmottier.com/2014/05/20/custom-animations-with-fragments
[search form]: /images/blog/android/search_web.gif
[search form mp4]: http://cyrilmottier.com/media/2014/05/custom-animations-with-fragments/search_web.mp4
[search android]: /images/blog/android/search_android.gif
[search android mp4]: http://cyrilmottier.com/media/2014/05/custom-animations-with-fragments/search_android.mp4
[search android slow]: /images/blog/android/search_android_slow.gif
[search android slow mp4]: http://cyrilmottier.com/media/2014/05/custom-animations-with-fragments/search_android_slow.mp4