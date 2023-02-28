---
layout: post
title:  "create my blog with github pages"
date:   2023-02-22 20:10:00
categories: jekyll github-pages
tags: jekyll github
---

* content
{:toc}

一直以来都想搭建一个自己的博客，之前比较忙，现在终于开始了，这篇文章记录一下自己搭建博客的过程以及遇到的问题。





### 摘要

本文旨在说明为何搭建个人博客，以及自己在搭建个人博客(github pages)的过程中遇到的问题。

### 为何搭建个人博客

​自从2021-02 迈入互联网行业之后，一直都有自己的[技术博客](https://www.cnblogs.com/zengzzzzz/)，当时为节省成本采用 oneNote + 博客园的方式，将自己在oneNote 上写的技术博客，导入到博客园进行管理。近期开始利用空余时间搭建自己的博客，至于搭建的原因也比较简单，主要有以下几点：希望能够统一在github管理自己的博客，可以集中精力在github上；看到别人的个人博客搞得不错，自己也就想着折腾一个试试，尝试一下新鲜事物；个人建站看起来比博客园更专业一些。

### 搭建个人博客中遇到的问题

#### 搭建方式选择

| 类型          | 优点                                                         | 缺点                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 博客园/csdn   | - 现在用的博客园，不需要改造<br>                                 | - 自定义程度太低 <br> - 与github分离，不满足自建的需求<br>            |
| 完全自建      | - 自定义程度高  <br>                                               | - 需要购买云服务器、域名等 <br> - 技术难度相比其他方案较高，时间成本高 <br> |
| neocities类似 | - 简单、傻瓜式部署 <br>  -可托管服务，不需要购买服务器、域名 <br>      | - 相比github pages 自定义化程度较低  <br>- 拖拽式部署，技术含量较低，是优点也是缺点 <br> - 与github 集成不太友好 <br> |
| netlify       | - 提供托管服务，不需要购买服务器、域名 - 可直接导入github respository，自定义workflow <br> - 国内访问较github pages 速度快，CDN 服务 <br> - 仅提供托管服务，网站可完全自定义 <br>| - 集成github 很友好，但是毕竟不是github pages <br> - 完全自定义情况下，时间成本较高 <br> |
| github pages  | - 提供托管服务，不需要购买服务器、域名 <br> - 可直接在github 上跑 workflow，实现推送即发布 <br> - 自定义程度较neocities 较高，但不及自建 <br> | - 国内访问速度较慢，无CDN <br> -自定义程度一般，主流仍采用  jekyll 或 hexo 生成静态html <br> |

综合考虑时间成本及github对其的友好程度，最终选择采用 github pages 方案，并在 jekyll 与 hexo 之间选择了github 官方推荐的 jekyll，可直接推送源码编译，更新blog。

### GITHUB PAGES 搭建问题

#### [官方教程](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)


按照官方教程，构建出最简主题 minma，其主要技术栈采用 ruby+bundle+jekyll ，自己还是比较陌生的，也是一边使用，一边学习，遇到不少诸如 gem add webrick 等问题，最终也构建出了minma。但是其最基本的分页功能都不存在，不过简单大概率意味着稳定，出现问题的概率也会低很多。

![image-20230227193934377](/img/2023-02-27-minma.png)

#### 主题选择

由于 minma 实在是简单到过分，自己考虑选几个jekyll theme 来美化一下，遂有了以下的踩坑经历。

##### [centarium](https://github.com/bencentra/centrarium)

在google找到的比较符合自己品味的jekyll free theme，主题确实比较好看，但是在构建过程中，发现其使用到了jekyll-archives 的插件，然而 github pages 托管服务出于安全的考虑，只支持了很少一部分的 [jekyll插件](https://pages.github.com/versions/) ,该插件并不被支持，导致blog 在github pages 上显示总存在问题。后来又通过google，找到在github pages 上绕过安全检测使用插件的[方法](http://ixti.net/software/2013/01/28/using-jekyll-plugins-on-github-pages.html)，在解决掉插件的问题后，部署完成后，自己感觉流程过于复杂，推送代码都要写个 rake task， 不易维护，遂抛弃该方案。

![image-20230227195334526](/img/2023-02-27-centarium.png)

##### [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)

由于在 centarium 中吸取到经验，不再选择需要额外插件的jekyll theme，于是选择了采用与 minimal 相同的 minimal mistakes，最小错误主题，简化流程，在使用过程中体验确实可以，但是主要存在的问题在于可配置化选项太多了，需要自己从零开始写配置项，还不如自己写html。并且使用remote theme 加载，导致会很慢。由于自己艺术水平确实有限，在手动搭配几次后，发现效果很一般，犹如淘宝买家秀的即视感，就抛弃该方案了。

![image-20230228115010622](/img/2023-02-07-minma-mistakes.png)

##### [huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)

由于minimal-mistakes 方案的中需要自己从零配置的问题，于是在github上搜索相关带有 github.io 的项目，便在其中找到了该 高star theme，确实令人惊艳，自己便将其下载到本地，阅读其实现代码，发现blog虽好，但是代码实现过于复杂，出现意外的bug，自己很难维护，毕竟自己平时工作中接触的前端项目还是比较少的。在本地测试和推倒github上进行测试，都取得了很好的效果，一方面是出于可维护性的考虑，另一方面也是感受该blog风格与自己并不相符，自己追求的是simple and strong，并不需要过多的装饰。

![image-20230228124712952](/img/2023-02-07-hux-blog.png)

##### [gaohaoyang.github.io](https://github.com/Gaohaoyang/gaohaoyang.github.io)

在决定放弃huxpro方案之后，开始在github上搜索其他类似项目，大概看了有20多个blog（不一一列出了），之后决定选择采用 gaohaoyang.github.io，选择他的原因也很简单，配置方便，实现简单，没有那么多花里胡哨的东西，实现代码基本都可以看懂，也能改得动；界面设计也符合自己的诉求，简洁实用。由此看来，自己确实也是个 simple is the best 主义者。

![image-20230228125342688](/img/2023-02-07-haoyang-blog.png)

### 反思

在整个搭建博客的过程中，周末折腾了俩天，再加上周一的上午，总算是比较满意了，其中过程还是比较曲折的，不过也是比较有成就感的，毕竟向个人建站迈出了第一步。总的来说，在此次搭建网站的过程中，感受到比较明显的一个变化，就是自己对于简洁与稳定性的追求占比变高了。几天时间内，对ruby bundle jekyll 这类建站技术栈也有了初步的认识，算是不错的收获。在接下来的时间，要研究一下如何将oneNote及博客园文章迁移到md文档了。千淘万漉虽辛苦，吹尽黄沙始到金，做一个长期主义者。

### 致谢

最后，非常感谢 [Gaohaoyang](https://github.com/Gaohaoyang) 提供的博客模板。 

