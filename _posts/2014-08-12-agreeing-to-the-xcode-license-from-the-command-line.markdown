---
title: "安装 Xcode 后命令行提示错误"
date: 2014-08-12 21:37:31 +0800
comments: true
categories: [Xcode]
desc: 在安装完新版本的 Xcode 后，在 iTerm 中执行命令总会有这样的提示：Agreeing to the Xcode/iOS license requires admin privileges, please re-run as root via sudo.
---

在安装完新版本的 Xcode 后，在 iTerm 中执行命令总会有这样的提示：

> Agreeing to the Xcode/iOS license requires admin privileges, please re-run as root via sudo.

这个时候执行 ``sudo xcrun cc`` 就可以显示 apple license ，输入 **agree** 即可。

参考: [Agreeing to the Xcode license from the command line][1]

[1]: http://blog.tomhennigan.co.uk/post/62238548037/agreeing-to-the-xcode-license-from-the-command-line