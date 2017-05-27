---
title: Timer(Task) ？ Not in Android ！
date: 2014-06-26 20:55:04 +0800
comments: true
categories: [Android]
desc: 我们对于 java.util.Timer(Task) 这两个类可谓不陌生，定时任务处处可见它俩如胶似漆的背影。只是，有心人必定会发现在 Android UI 中使用，它们带来的麻烦远远大于便利。我在多个场景碰到过这个问题，往往是忍气吞声，不厌其烦的处理着它们的「身后事」，还得提防它们时而罢工的臭毛病。
---

## 前言

我们对于 ``java.util.Timer(Task)`` 这两个类可谓不陌生，定时任务处处可见它俩如胶似漆的背影。只是，有心人必定会发现在 **Android UI** 中使用，它们带来的麻烦远远大于便利。

我在多个场景碰到过这个问题，往往是忍气吞声，不厌其烦的处理着它们的「身后事」，还得提防它们时而罢工的臭毛病。

## 解决方案

其实使用 Android 自己的 [Handler][1] 就可以解决这个问题。而且有如下优点：

- 重用

可以很好的重用我们的定时任务，而以往使用 ``Timer(Task)`` ，我们不得不销毁，再 ``new Timer(Task)``，不得不说是一种很粗鲁的方式。

- 可控性

``Timer(Task)`` 的组合更像是离弦之箭，不能收发自如。而使用 ``Handler`` 的方式，调用 ``removeCallbacks`` 即可轻松暂停或者重置任务。

### Timer

使用 [Handler][1] 来替代 [Timer][5]。

```java
Handler timerHandler = new Handler()
```

### TimerTask

简单可重用的 ``Runnable`` ，用来执行定时任务，但是这里是用的比较 **Hack** 的方式来实现循环的 Timer 功能的。

```java
CustomTimerTask timerTask = new CustomTimerTask();

class CustomTimerTask implements Runnable {
    @Override
    public void run() {
        // scheduled tasks

        // relaunch the task
        timerHandler.postDelayed(this, TIME_INTERVAL);
    }
}
```

### 操作定时任务

我们可以轻松的启动或者停止这个定时任务。

```java
timerHandler.postDelayed(timerTask, TIME_INTERVAL); // launch the task
timerHandler.removeCallbacks(timerTask);	// stop it
```

## 参考

[Timer(Task) = bad! Do it the Android way: Use a Handler :)][3]

[Updating the UI from a Timer][4]

[1]: https://developer.android.com/reference/android/os/Handler.html
[2]: https://developer.android.com/reference/android/os/Handler.html#removeCallbacks(java.lang.Runnable)
[3]: http://www.mopri.de/2010/timertask-bad-do-it-the-android-way-use-a-handler/
[4]: http://docs.huihoo.com/android/2.1/resources/articles/timed-ui-updates.html
[5]: https://developer.android.com/reference/java/util/Timer.html
