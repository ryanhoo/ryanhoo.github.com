---
title: Git 从其它分支提取指定文件
date: 2014-07-11 11:38:28 +0800
comments: true
categories: [Git]
desc: 在 Git 多个分支之间，使用 merge 或者 rebase 可以合并分支内容，非常方便，但是在特殊情况下，我需要从另外一个分支拉取一个或者某个指定的文件(夹)，手动 copy 代码当然可以，但是身为大程序猿，何须做这么接地气的事情？使用 git checkout 就可以轻松做到。
---

在 Git 多个分支之间，使用 **merge** 或者 **rebase** 可以合并分支内容，非常方便，但是在特殊情况下，我需要从另外一个分支拉取一个或者某个指定的文件(夹)，手动 copy 代码当然可以，但是身为大程序猿，何须做这么接地气的事情？使用 ``git checkout`` 就可以轻松做到。

## 应用场景

### A

我在 feature/A 分支下工作，在即将完成的时候，接到 PM 老爷的通知，这个功能要大改，基本就报废了。重新开一个 branch ，发现新需求下，model 定义还是一样的，而我只需要 model 下那几个文件。

### B

前几日用 NDK ，编译 .so 文件的时候，不知道什么问题，将 armeabi 目录下其它的 .so 文件都删除掉了，看到 git commit message 里提示删除，但是项目跑通没问题，当时就没在意。过了两天测试发现地图老是 crash ，发现了原因那就好办了~

不过万一其它分支也没有这个文件肿么办？不要紧，``git reset --hard [delete commit hashcode]`` 历史回溯到那个删除的节点，然后 在那个节点 folk 一个新分支，再将当前分支更新 ``git reset  --hard origin/[remote branch]`` ，在按照上面提到的思路执行就 ok 了。

## 从其它分支提取文件

```
git checkout [branch] -- [file name]
```