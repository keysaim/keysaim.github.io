---
layout:     post
title:      "如何给自己的博客网站加入评论系统"
subtitle:   ""
date:       2017-08-16
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - blog
    - 教程
---

# 前言

在[这一篇](https://keysaim.github.io/2017/08/15/how-to-setup-your-github-io-blog/)博文中，咱们介绍了如何快速的搭建个人的博客网站，但是这个博客网站是基于[Github Pages](https://pages.github.com/)的纯静态网站，自身是不带任何的可交互的元素的，自然也就没有评论系统。但是，对于一个好的博客网站，如果没有评论系统，那基本上就属于自娱自乐了，也违背了博客分享的精神。因此，本文将着重介绍如何快速的给自己的博客网站加入评论系统。

# 评论系统介绍

所谓评论系统，相信对于关注博客的你来说肯定不陌生。通常来讲有两大类，一类是网站自己提供评论系统，这个在很多较大的博客平台比较常见；一类是基于第三方的评论系统，这个在个人博客网站较为常见。对于个人博客，由于很多都是简单的托管在其他平台上面，比如博主网站托管于Github，因此很多都基本上是静态的网站，自身是没法提供评论系统。也正因为如此，也就催生了非常多的第三方的评论系统。这些系统国外比较有名的是[Disqus](https://disqus.com/)，国内更加五花八门了。国外的由于天朝特殊的网络环境，并不十分稳定，所以对于国内的博客网站来说，通常都会选择国内的第三方评论系统。至于国内的评论系统有哪些选择呢，建议参考[这篇博文](https://www.mokeyjay.com/archives/1732)。总的来说，由于评论系统这个东东，目前还没有明显的盈利的地方，倒闭关门是很有可能的，因此尽量选择大公司为后台的吧。

最近在研究评论系统的时候，偶然发现[有人](http://hydroecology.net/using-github-to-host-blog-comments/)居然直接使用Github的Issue系统来做评价系统，顿时眼前一亮啊。鉴于咱们的博客便是基于Github的，如果可行，那简直绝配了。于是博主就开始疯狂的搜索有关现状，Github都多少年了，博主深信应该已经有人造过轮子了。这类轮子肯定是属于前端方面的，于是在[npm](https://www.npmjs.com/)网站上面搜索，果然[发现不少](https://www.npmjs.com/search?q=github%20issue%20comment&page=1&ranking=popularity)。比较下来，发现一款国人开发的[gitment](https://www.npmjs.com/package/gitment)最为完整。这也就本文要介绍的评论系统了，本博客网站在可见的未来将采用该评论系统，这里对作者的分享表示感谢。

# gitment使用

gitment的使用在[官网](https://imsun.net/posts/gitment-introduction/)有较为详细的介绍，步骤写的还是比较清楚的。这里只是对于讲述可能踩到的坑：

* 请务必不要使用你的`<username>.github.io`这个repo作为`gitment`的repo

    可能这个repo作为Github特殊的repo，使用它作为`gitment`的repo时，总是不能够认证成功。建议再创建一个专门用来存放评论的repo。

* [生成oauth授权](https://github.com/settings/applications/new)的时候，请务必确保`Authorization callback URL`是你的博客网站，如博主的是`https://keysaim.github.io`。否则会授权失败。


* 每次提交博文的时候，请务必记得初始化改博文的评论，否则读者无法进行评论

    所谓初始化其实就是在你的博文下面用自己的Github账号登陆之后，会出现一个`Initialize Comments`字样的按钮，点击该按钮完成初始化。改初始化其实就是在你的Github repo里面针对这篇博文生成一个新的Issue。

# 结语

`gitment`之类的评论系统是基于Github的Issue系统，每篇博文都对应于你的Github repo里面的一个Issue，博文里面的评论其实都是在改Issue里面的评论，因此，你是有绝对权限对所有评论进行管理的。另外，由于`gitment`项目开始不久，并且貌似作者比较忙，因此功能性，质量方面还是存在不足，当你使用过程中如果遇到任何问题，可以直接去[gitment repo](https://github.com/imsun/gitment)里面查看一下是不是别人也碰到类似问题。可能的情况下，你也可以提交自己的解决方案。
