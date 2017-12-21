---
layout:     post
title:      "如何快速搭建自己的github.io博客"
subtitle:   ""
date:       2017-08-15
author:     "keysaim"
header-img: ""
catalog: true
tags:
    - github
    - 教程
---

# 闲聊一下

在这知识剧烈膨胀的时代，如何记录、整理、分享自己的所学所感无疑显得十分重要，而博客便是最好的方式之一。现在已经有了各式各样的博客平台，有基于第三方的平台的（如[博客园](https://www.cnblogs.com/)等），也有自己搭建的（如基于[Ghost](https://ghost.org/)等，当然，也有很多干脆自己动手DIY了）。总之，博客的世界已然丰富多彩，留给咱们更多的不是有没有，而是哪个好。这里咱不讨论博客哪家强的问题，只推荐一款博主觉得不错的选择：`github.io`。

要问全球最大的基佬交友网站是哪个，我相信不少答案必须是咱们的[github](https://github.com)。而`github.io`便是其出品，品质必须是有保证的，最重要的一点是基于github的repo管理，这意味着咱们对其是有觉得的控制，这个跟放在第三方的平台比，可控性要好太多。下面咱们将详细讲述如何基于`github.io`打造属于自己的博客网站。

要完成自己的`github.io`博客网站，总共分三步：

* 开通自己的`github.io` repo
* 选择一款Jekyll的主题
* 编写发布博客

# 开通自己的`github.io` repo

`github.io`是完全基于github创建的，其本质上是在你的github账户下创建一个特殊的repo。你可以参照[如下步骤](https://pages.github.com/)完成：

* 创建repo

    当然，一切的前提是你得首先有个github的账户，这里还请自行解决。登陆你的账户后，你可以[创建一个新的repo](https://github.com/new)。***请务必注意该repo的名字，必须保持格式`<username>.github.io`，其中`<username>`替换成你的github账户名***，这里假定创建的repo为`tobiasalin.github.io`

    <div style="text-align: center">
        <img src="https://pages.github.com/images/user-repo@2x.png" alt=""/>
    </div>

* 把你创建的repo clone到本地

    本文假定你已经有一定的`git`使用基础了，如果没有也没关系，Google一下，`git`的基本使用极为简单。

    ```
    $ git clone https://github.com/tobiasalin/tobiasalin.github.io
    ```

* 编写简单的博客首页

    ```
    $ cd tobiasalin.github.io
    $ echo "Hello World!" > index.html
    $ git add index.html
    $ git commit -m "Init commit"
    $ git push origin master
    ```
* 打开博客网站`https://<username>.github.io`

    不出意外，你就可以看到你的`Hello World!`博客首页了。如果不小心出了意外，通常情况下，你只需等一会再刷新就会好，要是还没好，通常说明你的运气实在太背，请自行了断。

# 选择一款`Jekyll`的主题

`github.io`默认采用`Jekyll`作为建站工具。[Jekyll](https://jekyllrb.com/)是一款当前火热的开源的静态网站建站工具，拥有非常庞大的使用群里和社区，其[Github](https://github.com/jekyll/jekyll)截止本文，已经有超过3W+的star，拥有丰富的插件，丰富的主题，并且有无数的人已经帮你早出了无数的轮子可供参考。`Jekyll`自身的强大功能已经足够你打造自己心仪的静态网站（这里注意的是`静态网站`，`Jekyll`没有任何的后台数据库），然而前提是你自己还是得有一定的前端功底，而为了不至于长的太难看，你还得有一定的设计能力。这一下子把大部分人给难住了，咱们只是为了单纯的写写博客啊，至于有这么多要求吗？看到这里，很多人可能觉得此法不怎么方便啊，然则，正如刚刚反复强调的，`Jekyll`已经有一个非常庞大的社区，这就意味着，你完全可以借鉴别人已经造好的轮子，放在`Jekyll`这里，咱们应该成为主体（Theme）比较合适。本文推荐国内用户可以考虑一款[国人开发的主题](https://github.com/Huxpro/huxpro.github.io)。***本博客即是采用了这个主题***。

* Fork出自己的repo

    为了便于管理，建议先把[Huxpro](https://github.com/Huxpro/huxpro.github.io) fork到自己的账户下

* clone主题

    ```
    $ git clone git@github.com:keysaim/huxpro.github.io.git
    ```

    当然，你也可以直接clone它的样板repo

    ```
    $ git clone git@github.com:Huxpro/huxblog-boilerplate.git
    ```

* 添加自己的github.io repo

    clone了Huxpro的repo之后，还需要将自己的github.io repo添加到Huxpro repo中，以方便后面讲修改同步到自己的repo中：

    ```
    $ git remote add mine https://github.com/tobiasalin/tobiasalin.github.io
    ```

    其中，`mine`是你自己repo的别名，当然，你完全可以用其它任何合法的名字。

* 修改必要的配置

    clone之后的repo其实是Huxpro自己的博客网站，里面有非常多作者自己的博文，可根据自己的需要进行必要的删减。基于`Jekyll`的博客网站，对于配置，非常重要的一个文件是`_config.yml`文件，代开这个文件进行必要的修改：

    ```yaml
    # Site settings
    title: 窗外蟋蟀
    SEOTitle: 窗外蟋蟀的博客 | Keysaim Blog
    header-img: img/home-bg-hill.jpg
    email: keysaim@gmail.com
    description: "描述"
    keyword: "窗外蟋蟀, keysaim"
    url: "https://keysaim.github.io"
    baseurl: ""
    ...
    ```

# 编写发布博客

`Jekyll`对于博文，都是要求放在`_posts`目录下面，同时对博文的文件名有[严格的规定](https://jekyllrb.com/docs/posts/#creating-post-files)，必须保持格式`YEAR-MONTH-DAY-title.MARKUP`，通常情况下，咱们采用推荐的`Markdown`撰写博文，基于该格式，本博文的文件名为`2017-08-15-how-to-setup-your-github-io-blog.md`。

写好博文之后，就可以通过`git`提交博文了：

```
$ git add _posts/2017-08-15-how-to-setup-your-github-io-blog.md
$ git commit -m "Add how to setup your github.io blog"
$ git push mine master
```

*请务必注意，这里提交的repo是`mine`，也就是你自己的github.io repo。*等一会（通常几秒到几十秒不等），就可以打开自己的博客网站查看博文了，这里是我的博客网站`https://keysaim.github.io`。

## 本地查看自己的博客

有时候，在提交到`github`之前，咱们总想先看看博文的效果如何，既然`github`采用的也是`Jekyll`，那么咱们完全可以采用`Jekyll`在本地构建网站，查看博文效果。

* 按照`Ruby`（请务必确保版本在`1.9.3`以上）

    可以参照[官网教程](https://www.ruby-lang.org/en/downloads/)进行安装，这里，如果你使用的是Mac，可以安装如下：

    ```
    $ brew install ruby
    ```

* 安装`Github pages`

    `Github pages`其实就是`github`基于`Jekyll`用来构建`github.io`的工具，安装好Ruby之后可以执行：

    ```
    $ gem install github-pages
    ```

* 开启`Jekyll`本地服务

    ```
    $ cd keysaim.github.io
    $ jekyll serve --watch
    ```

    默认情况下，该服务会侦听在本地`4000`的端口上，可以打开浏览器访问`http://127.0.0.1:4000`，这样就可以在本地查看自己的博文效果了。

# 结语

`github.io`通过基于`Jekyll`工具的[Github pages](https://pages.github.com/)来自动构建网站，同时本身又是`github`的repo，为使用者提供完全的内容控制，十分便利灵活。当然，要用好这个工具，还是对你有一定的要求：

* 必须得有基本的`git`使用基础

* 必须对前端有一定的概念

* 必须较为熟悉`Markdown`撰写

不过，幸运的是，这三点不论哪一点，其实都是相对较为简单的，`git`的基本使用，Google一下，我相信1小时学会不是太大问题；对前端的要求，由于前人已经造了足够多的轮子，咱们完全可以先借鉴别人的，等以后慢慢熟悉起来之后再考虑自己DIY；对于`Markdown`，个人觉得其语法真的是非常的简单，只要静下心来，一个下午足够让你编写基本的文章了。

最后，这里再次感谢[Huxpro](https://github.com/Huxpro/huxpro.github.io)提供本站的`Jekyll`主题。

# 参考

* [Build A Blog With Jekyll And GitHub Pages](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/)
* [Github pages](https://pages.github.com/)
