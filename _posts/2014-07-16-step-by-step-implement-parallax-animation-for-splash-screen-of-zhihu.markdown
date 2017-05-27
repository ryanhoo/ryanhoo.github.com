---
title: "[Parallax Animation]实现知乎 Android 客户端启动页视差滚动效果"
date: 2014-07-16 18:22:58 +0800
comments: true
categories: [Android, Animation]
desc: Parallax Scrolling (视差滚动)，是一种常见的动画效果。视差一词来源于天文学，但在日常生活中也有它的身影。在疾驰的动车上看风景时，会发现越是离得近的，相对运动速度越快，而远处的山川河流只是缓慢的移动着，这就是最常见的视差效果。视差动画独有的层次感能带来极为逼真的视觉体验，iOS、Android Launcher、Website 都将视差动画作为提升用户视觉愉悦度的不二选择。本文就知乎 Android 客户端启动页面为例，教你如何实现视差滚动效果。
---

* Contents
{:toc}

> 欢迎转载，但请务必注明出处！
> 
>[http://ryanhoo.github.io/blog/2014/07/16/step-by-step-implement-parallax-animation-for-splash-screen-of-zhihu/][6]

# 前言

``Parallax Scrolling (视差滚动)``，是一种常见的动画效果。视差一词来源于天文学，但在日常生活中也有它的身影。在疾驰的动车上看风景时，会发现越是离得近的，相对运动速度越快，而远处的山川河流只是缓慢的移动着，这就是最常见的视差效果。视差动画独有的层次感能带来极为逼真的视觉体验，**iOS**、**Android Launcher**、**Website** 都将视差动画作为提升用户视觉愉悦度的不二选择。

客户端应用第一次打开出现引导页也不是什么新鲜的事儿，``ViewPager`` 配上几张设计师精心绘制的图片，分分钟即可了事。但是总有人把平凡的事情做到不平凡，如本文的知乎客户端，亦或是新浪微博贺岁版，百度贴吧某版等众多应用里都出现了视差动画的身影，随着用户手指的滑动，反馈以灵动、贴近真实的视觉以及操作体验，对应用的初始印象登时被提升到一个极高的点。

给我印象最深的是去年新浪微博的贺岁版，引导页是一系列的年画，里面有红色剪纸的小孩儿，滑动界面的时候感觉这些元素在「动」，是真正的灵动，能勾起人童年的回忆，年味儿十足。不过话说我年怎么过跟新浪微博一毛钱关系都没有，但是这个启动页却是深得我意。只是这个版本的微博找不到了，正好前两天看到知乎的启动页做的也不错，就正好拿来练练手吧。

本文就知乎 Android 客户端启动页面为例，教你如何实现视差滚动效果。

![知乎 Android 客户端启动页][1]

# 界面分析

细心把玩下知乎的启动页，不难分析出来，视差动画主要体现在背景层渐变、内容层元素差异滚动上，动画内容分别是：

- 内容：元素差异滚动，形成视差效果(*)
- 背景：随着界面的滑动，颜色由深蓝色渐变为浅蓝色(*)
- 文字：底部提示文案会随页面变动而切换，有简单的淡入淡出效果
- 界面动画：界面打开，元素的出场动画（第一页以及最后一页）

鉴于其它几项比较简单，本文主要讲视差动画以及背景渐变的实现，其它几项请自行参阅代码，见后文。

# Parallax Scrolling

这里的视差滚动效果，主要表现为内容元素滚动速率的差异上。比如在 ``ViewPager`` 中滑动了 ``1px`` ，而 **A** 元素移动 ``2px`` ， **B** 元素移动 ``1.5px`` ，这种移动差距的比率，我称之为 ``parallaxCofficient`` ，即 **视差系数** 或者 **视差速率** ，正是同一个界面中的元素，由于层级不同，赋予的视差系数不同，在移动速度上的差异形成了视差的错觉，这就是我们要追求的效果。

那知道原理就好办了，使用 ``ViewPager.OnPageChangeListener`` ，动态计算不就得了。 no no no ! 后面完成背景渐变效果确实需要计算这个，但是 ``ViewPager`` 已经为我们准备好了变形元素 **transformium** : ``ViewPager.PageTransformer`` ，它有一个抽象方法 ``transformPage(View page, float position)`` ，正是为我们完成视差动画量身定制的。

## ViewPager.PageTransformer

[PageTransformer][2] 在 ``ViewPager`` 滑动时被触发，它为我们自定义页面中进行视图变换打开了一扇大门。

```java
public abstract void transformPage (View page, float position)
```

在 ``ViewPager`` 源码中，我们可以很直观的看到它的调用过程：

```java
// ViewPager#onPageScrolled
if (mPageTransformer != null) {
    final int scrollX = getScrollX();
    final int childCount = getChildCount();
    for (int i = 0; i < childCount; i++) {
        final View child = getChildAt(i);
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();

        if (lp.isDecor) continue;

        final float transformPos = (float) (child.getLeft() - scrollX) / getClientWidth();
        mPageTransformer.transformPage(child, transformPos);
    }
}
```

#### Param 1: View page

从上面的代码中，不难看出，``page`` 就是当前被滑动的页面，调试得知，每一个 **child view** 被 ``NoSaveStateFrameLayout`` 包装，也就是说 ``page.getChildAt(0)`` 即是每个 ``page`` 实际的 **child view** 。

#### Param 2: float position

``position`` 这个参数不看代码或者文档，总会误以为就是我们熟知的 ``integer position`` ，不过它实际上是滑动页面的一个相对比例，本质跟 1、2、3、4 这种 ``position`` 是一样的。

比如知乎启动页共有 **6** 个页面，分别是 **A**、**B**、**C**、**D**、**E**、**F** 初始状态也就是 **A** 页面静止时，**A** 页面的 ``position`` 正好是 **0** ，**B** 页面是 **1** 。而后滑动页面（B -> A），在这个过程中 **A** 的 ``position`` 是间于 ``[-1, 0]`` ，**B** 页面则是间于 ``[0, 1]`` 。

不过这个参数的文档却是简单不够直观，对照上面的例子，现在应该很清晰了。

> Position of page relative to the current front-and-center position of the pager. 0 is front and center. 1 is one full page position to the right, and -1 is one page position to the left.

## ParallaxTransformer

根据上面的分析，我们可以得出一个相对简单的自定义 **transformer** ，对 ``page view`` 进行遍历，递增或者递减其 ``parallaxCofficient`` ，以得到我们预期的效果，具体的系数设置请参考代码。

```java
class ParallaxTransformer implements ViewPager.PageTransformer {

    float parallaxCoefficient;
    float distanceCoefficient;

    public ParallaxTransformer(float parallaxCoefficient, float distanceCoefficient) {
        this.parallaxCoefficient = parallaxCoefficient;
        this.distanceCoefficient = distanceCoefficient;
    }

    @Override
    public void transformPage(View page, float position) {
        float scrollXOffset = page.getWidth() * parallaxCoefficient;

        // ...
        // layer is the id collection of views in this page
        for (int id : layer) {
            View view = page.findViewById(id);
            if (view != null) {
                view.setTranslationX(scrollXOffset * position);
            }
            scrollXOffset *= distanceCoefficient;
        }
    }
}
```

# 背景渐变

留心才会发现，从第一页滑动到最后一页，背景色会平滑的从深蓝色过度到浅蓝色，这种效果又该怎么实现呢？

用过 ``Property Animation`` 的同学应该知道，以前的 ``Animation`` 只能用在 **View** 上，而 ``Property Animation`` 却可以用在任意类型属性值上，这归功于 ``TypeEvaluator`` 。

正好我们有 ``ArgbEvaluator`` ，它可以估算两个颜色值之间，任意部分的色值。因此，只需要指定起始色值以及最终的色值，传入滑动所对应的 ``fraction`` 即当前位置相对总距离的比例值，即可获得相应的色值。

```java
public class ArgbEvaluator implements TypeEvaluator {

    public Object evaluate(float fraction, Object startValue, Object endValue) {
    	// ...
    }
}
```

当然，前面说到需要使用 ``ViewPager.OnPageChangeListener`` 的：

```java
class GuidePageChangeListener implements ViewPager.OnPageChangeListener {

    ArgbEvaluator mColorEvaluator;

    int mPageWidth, mTotalScrollWidth;

    int mGuideStartBackgroundColor, mGuideEndBackgroundColor;

    public GuidePageChangeListener() {
        mColorEvaluator = new ArgbEvaluator();

        mPageWidth = getWindowManager().getDefaultDisplay().getWidth();
        mTotalScrollWidth = mPageWidth * mAdapter.getCount();

        mGuideStartBackgroundColor = getResources().getColor(R.color.guide_start_background);
        mGuideEndBackgroundColor = getResources().getColor(R.color.guide_end_background);
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        float ratio = (mPageWidth * position + positionOffsetPixels) / (float) mTotalScrollWidth;
        Integer color = (Integer) mColorEvaluator.evaluate(ratio, mGuideStartBackgroundColor, mGuideEndBackgroundColor);
        mPager.setBackgroundColor(color);
    }

    @Override
    public void onPageSelected(int position) {}

    @Override
    public void onPageScrollStateChanged(int state) {}
}
```

# 源码

代码已经 **push** 到 **Github** 了，诸位自取。不过请注意，其素材均取自于知乎 Android 客户端（你懂的），学习交流即可，请勿用作商业用途。

还求更优雅的实现方式，欢迎发起 ``pull request`` 。

## Github : [Zhihu-Parallax-Animation][3]

# 参考

1. [Using Parallax for Fun and Profit][4]
2. [视差滚动(Parallax Scrolling)效果的原理和实现][5]

[1]: /images/blog/android/zhihu-parallax-animation.gif
[2]: https://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html
[3]: https://github.com/ryanhoo/Zhihu-Parallax-Animation
[4]: http://blog.neteril.org/blog/2014/01/02/using-parallax-for-fun-and-profit/
[5]: http://www.cnblogs.com/JoannaQ/archive/2013/02/08/2909111.html
[6]: http://ryanhoo.github.io/blog/2014/07/16/step-by-step-implement-parallax-animation-for-splash-screen-of-zhihu/