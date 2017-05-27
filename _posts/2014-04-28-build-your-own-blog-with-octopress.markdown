---
title: "使用Octopress搭建自己的博客"
date: 2014-04-28 22:07:27 +0800
comments: true
categories: [Octopress]
desc: 一直想搭建一个自己的博客，开始付诸行动！刚刚接触到 Octopress ，总体感觉还是不错的，但是它不像 segmentfault 、 oschina 之类的平台，什么都具备，很多东西默认没有，需要自己配置，不过在玩的过程中总是能学到新东西的。
---

* Contents
{:toc}

一直想搭建一个自己的博客，开始付诸行动！

刚刚接触到Octopress，总体感觉还是不错的，但是它不像segmentfault、oschina之类的平台，什么都具备，很多东西默认没有，需要自己配置，不过在玩的过程中总是能学到新东西的。

## Step 1: 安装Octopress

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
bundle update    # 安装依赖的组件
rake install     # 安装默认的Octopress主题
```

## Step 2: 配置

```
git remote rm origin
git remote add origin git@github.com:ryanhoo/ryanhoo.github.com.git
git remote add octopress git://github.com/imathis/octopress.git  # 为了octopress的升级而添加
```

## Step 3: 设置github pages

在github上创建一个仓库，注意仓库名称要以下这种格式 yourname.github.com，这样代码发布后自动这个url就可以访问了（此处一定要注意哦，我刚开始没注意，死活没得到想要的效果）。 例如你的 GitHub 帐号是 jack 就将 Repository 命名为 jack.github.com， 完成后会得到一组 GitHub Pages URL http://yourname.github.com/ (注意不能用 https协议，必须用 http协议)。

```
rake setup_github_pages
```

## Step 4: 编译、预览与发布

```
rake generate  			# 生成网页
rake preview   			# 预览
rake deploy    			# 发布
rake gen_deploy 		# 相当于生成+发布
rake new_page["name"] 	# 创建新页面
rake new_post["name"]	# 新建博文
```

## Step 5: 更新 Octopress

```
git remote add octopress git://github.com/imathis/octopress.git
git pull octopress master     # Get the latest Octopress
bundle install                # Keep gems updated
rake update_source            # update the template's source
rake update_style             # update the template's style
```

## 参考

- [主题配置][1]
- [增加新浪微博链接][2]
- [Liquid Basics][3]
- [添加目录][4]

[1]: https://github.com/shashankmehta/greyshade
[2]: http://imallen.com/blog/2013/05/12/add-support-for-weibo-and-dribbble-to-greyshade.html
[3]: http://docs.shopify.com/themes/liquid-basics
[4]: http://khaos.github.io/blog/2012/12/05/generating-toc-in-octopress/