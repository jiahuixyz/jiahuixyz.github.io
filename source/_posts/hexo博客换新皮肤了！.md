---
title: hexo博客换新皮肤了！
date: 2023-05-30 01:07:34
tags:
---

很久没打理博客了，最近准备写些文章，结果clone下来发现hexo都不能正常install了，原因是node版本有些高，和3.x的hexo不兼容，那么，索性这次就把hexo版本升级到最新的6.x，然后给博客换一个新主题。

具体思路是，下载新版的hexo-cli，用hexo init初始化一个文件夹，然后挑选自己喜欢主题，clone到themes下，接着将旧的post文件夹下的文章全部搬过来，然后呢，就把博客在本地启动起来，左看看、右看看，哪里有不合适的就改改配置，或者改改主题的源码。

改的差不多了，还有最重要的一步，配置一下自动部署到GitHub Pages。上次是用的一个叫hexo-deployer-git的hexo插件，能够实现一键部署，原理就是本地生成好静态文件，直接上传，这次打算换一种方式，用一下GitHub的Actions，具体做法也很简单，在.github/workflows下建一个ci文件就行了，脚本官网上已经写好了。

<!--more-->

好了，最后对比下博客前后的变化吧～

这个是旧的==>

![](https://jiahuixyz.github.io/img-service/img/old_blog.png)

这个是新的==>

![](https://jiahuixyz.github.io/img-service/img/new_blog.png)

怎么说呢，算是不同时期的喜好不一样吧，旧的主题是那种特别朴素的那种，直接使用了文章归档页当了主页，没有图片和装饰。新的博客主题可以自定义一张图片，这里我选了一张很喜欢的下雨打伞的图片，有了图片博客看着就比较自然了，平易近人的感觉，然后主页的文章也不只是显示标题了，还可以展示预先设定的文章摘要（摘要需要在文章里添加`<!--more-->`），说到这里要感谢下hexo-theme-paperbox和landscape-plus两个主题的作者sun11、xiangming，感谢你们做出了这么优秀的主题！

好了，这篇小记就到此结束了～