---
title: 【译】Android：更好的自定义字体方案
date: 2014-05-05 17:48:26 +0800
comments: true
categories: [Android]
desc: 在一个应用中，我需要在所有的 UI 组件中使用客户提供的字体。这听起来似乎是个很稀松平常的任务，不是吗？是的，我当时也是这么想的。然后我震惊了， Android 竟然没有提供一个简单优雅的方式来做这件事情! 所以，在这篇文章中我会展示Android提供的默认方法，然后我会分享更加简单优雅的解决方案。
---

> 原文：[http://vision-apps.blogspot.hk/2012/02/android-better-way-to-apply-custom-font.html][1]

在一个应用中，我需要在所有的UI组件中使用客户提供的字体。这听起来似乎是个很稀松平常的任务，不是吗？是的，我当时也是这么想的。然后我震惊了，Android竟然没有提供一个简单优雅的方式来做这件事情!

所以，在这篇文章中我会展示Android提供的默认方法，然后我会分享更加简单优雅的解决方案。

### 情景

你需要为整个应用替换自定义字体。

### 解决方案

#### 1）Android默认方法 #1

你可以通过ID查找到View，然后挨个为它们设置字体。在单个View的情况下，它看起来也没有那么可怕。

```java
Typeface customFont = Typeface.createFromAsset(this.getAssets(), "fonts/YourCustomFont.ttf");
TextView view = (TextView) findViewById(R.id.activity_main_header);
view.setTypeface(customFont);
```

但是在很多TextView、Button等文本组件的情况下，我敢肯定你不会喜欢这个方法的。:D

#### 2）Android默认方法 #2

你可以为每个文本组件创建一个子类，如TextView、Button等，然后在构造函数中加载自定义字体。

```java
public class BrandTextView extends TextView {

      public BrandTextView(Context context, AttributeSet attrs, int defStyle) {
          super(context, attrs, defStyle);
      }
     public BrandTextView(Context context, AttributeSet attrs) {
          super(context, attrs);
      }
     public BrandTextView(Context context) {
          super(context);
     }
     public void setTypeface(Typeface tf, int style) {
           if (style == Typeface.BOLD) {
                super.setTypeface(Typeface.createFromAsset(getContext().getAssets(), "fonts/YourCustomFont_Bold.ttf"));
            } else {
               super.setTypeface(Typeface.createFromAsset(getContext().getAssets(), "fonts/YourCustomFont.ttf"));
            }
      }
 }
```

然后只需要将标准的文本控件替换成你自定义的就可以了（例如BrandTextView替换TextView）。

```xml
<com.your.package.BrandTextView
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:text="View with custom font"/>
<com.your.package.BrandTextView
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:textStyle="bold"
         android:text="View with custom font and bold typeface"/>
```

还有，你甚至可以直接在XML中添加自定义的字体属性。要实现这个，你需要定义你自己的```declare-styleable```属性，然后在组件的构造函数中解析它们。

为了不占篇幅介绍这么基础的东西，这里有一篇不错的文章告诉你怎么自定义控件属性。

> [http://kevindion.com/2011/01/custom-xml-attributes-for-android-widgets/][2]

在大多数情况下，这个方法还不赖，并且有一些优点（例如，切换字体粗细等等，字体可以在组件xml文件的typeface属性中定义）。但是我认为这个实现方法还是太重量级了，并且依赖大量的模板代码，为了一个替换字体的简单任务，有点儿得不偿失。

#### 3）我的解决方案

理想的解决方案是自定义主题，然后应用到全局或者某个Activity。
但不幸的是，Android的```android:typeface```属性只能用来设置系统内嵌的字体，而非用户自定义字体(例如assets文件中的字体)。这就是为什么我们无法避免在Java代码中加载并设置字体。

所以我决定创建一个帮助类，使得这个操作尽可能的简单。使用方法：

```
FontHelper.applyFont(context, findViewById(R.id.activity_root), "fonts/YourCustomFont.ttf");
```

并且这行代码会用来加载所有的基于TextView的文本组件（TextView、Button、RadioButton、ToggleButton等等），而无需考虑界面的布局层级如何。


![标准(左)与自定义(右)字体的用法。][3]

这是怎么做到的？非常简单：

```java
public static void applyFont(final Context context, final View root, final String fontName) {
    try {
        if (root instanceof ViewGroup) {
            ViewGroup viewGroup = (ViewGroup) root;
            for (int i = 0; i < viewGroup.getChildCount(); i++)
                applyFont(context, viewGroup.getChildAt(i), fontName);
        } else if (root instanceof TextView)
            ((TextView) root).setTypeface(Typeface.createFromAsset(context.getAssets(), fontName));
    } catch (Exception e) {
        Log.e(TAG, String.format("Error occured when trying to apply %s font for %s view", fontName, root));
        e.printStackTrace();
    }
}
```

正如你所看到的，所需要做的仅仅是将基于TextView的文本组件从布局中遍历出来而已。

你可以在这里下载到示例代码，里面有[```FontHelper```][4]的具体用法。

### 译者注

在多个项目中，我都碰到过类似的需求，早期采用的是第二种实现方法，但是缺点在于对于第三方组件，你需要去修改别人的代码，才能实现自定义字体，这恰恰违反了OC（Open & Close）原则，对扩展开放，对修改封闭。

刚看到第三种的时候，也是惊为天人，姑且不说结果，我觉得这种活跃的思路非常重要，很值得学习参考。

但是最后被team里的人否决了，理由是违背组件设计原则，实现方式略嫌粗暴。后来我仔细想想，一个是要做好内存管理（似乎会引起内存问题），视图状态改变，也要重复加载（横竖屏、onResume等），也绝对不是简单的活儿。

所以暂定使用第一种方法，typeface使用单例，需要时设置字体。

我个人觉得第一种还是个体力活，而且到后来，这个代码重复率还是非常高的，这又违背了DRY原则。

在地铁上的时候，突然想到DI（Dependency Inject）。已经有一些DI的框架，如ButterKnife，那写出来应该是这样：

```
@CustomFont(R.id.textView) TextView textView
```
or

```
@InjectView(id=R.id.textView, customFont=true) View anyView
@InjectView(id=R.id.textView, customFont=true, font="fonts/ltfz.ttf") View anyView
```
这样写出来代码相比重复写setTypeface要好一些。

目前我们的项目还没有使用这类DI框架，等以后引入了，使用第二种注入，写起来应该是很爽的。

保持更新。
 
### 参考

- [DI框架][5]
- [ButterKnife][6]

[1]: http://vision-apps.blogspot.hk/2012/02/android-better-way-to-apply-custom-font.html
[2]: http://kevindion.com/2011/01/custom-xml-attributes-for-android-widgets/
[3]: /images/blog/2014-05-05-before and after.png
[4]: http://db.tt/i9S80Mgr
[5]: http://www.javacodegeeks.com/2014/02/dependency-injection-options-for-java.html
[6]: http://stormzhang.github.io/openandroid/android/2014/01/12/android-butterknife/