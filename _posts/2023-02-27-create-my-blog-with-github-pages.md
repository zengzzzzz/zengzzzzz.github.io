---
layout: post
title:  "create my blog with github pages"
date:   2023-02-22 20:10:00
categories: jekyll github-pages
tags: jekyll RubyGems
---

* content
{:toc}

一直以来都想搭建一个自己的博客，之前比较忙，现在终于开始了，这篇文章记录记录一下，自己搭建博客的过程,以及遇到的问题。





### 摘要

本文旨在说明为何搭建个人博客，以及自己在搭建个人博客的过程中遇到的问题。

### 为何搭建个人博客

​     自己自从2021-02 迈入互联网行业之后，一直都有在做技术博客的记录，当时为节省成本采用 oneOnte + 博客园的方式，将自己在oneOnte 上写的技术博客，导入到博客园进行管理。近期开始利用空余时间搭建自己的博客，至于搭建的原因也比较简单，主要有以下几点：希望能够统一在github管理自己的博客，可以集中精力在github上；看到别人的个人博客搞得不错，自己也就想着折腾一个试试，尝试一下新鲜事物；看起来比博客园更专业一些，好像也不一定。

博客园地址 https://www.cnblogs.com/zengzzzzz/

### 搭建个人博客中遇到的问题

#### 搭建方式选择

| 类型          | 优点                                                         | 缺点                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 博客园/csdn   | - 现在用的博客园，不需要改造                                 | - 自定义程度太低 - 与github分离，不满足自建的需求            |
| 完全自建      | - 自定义程度高                                               | - 需要购买云服务器、域名等 - 技术难度相比其他方案较高，时间成本高 |
| neocities类似 | - 简单、傻瓜式部署  -可托管服务，不需要购买服务器、域名      | - 相比github pages 自定义化程度较低 - 拖拽式部署，技术含量较低，是优点也是缺点 - 与github 集成不太友好 |
| netlify       | - 提供托管服务，不需要购买服务器、域名 - 可直接导入github respository，自定义workflow - 国内访问较github pages 速度快，CDN 服务 - 仅提供托管服务，网站可完全自定义 | - 集成github 很友好，但是毕竟不是github pages - 完全自定义情况下，时间成本较高 |
| github pages  | - 提供托管服务，不需要购买服务器、域名 - 可直接在github 上跑 workflow，实现推送即发布  - 自定义程度较neocities 较高，但不及自建 | - 国内访问速度较慢，无CDN -自定义程度一般，主流仍采用  jekyll 或 hexo 生成静态html |

综合考虑时间成本及github对其的友好程度，最终选择采用 github pages 方案，并在 jekyll 与 hexo 之间选择了github 官方推荐的 jekyll，可直接推送源码编译，更新blog。

#### GITHUB PAGES 搭建问题

#### 官方教程

官方教程：https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll

按照官方教程，构建出最简主题 minma，其主要技术栈采用 ruby+bundle+jekyll ，自己还是比较陌生的，也是一边使用，一边学习，遇到不少诸如 gem add webrick 等问题，最终也构建出了minma。但是其最基本的分页功能都不存在，不过简单大概率意味着稳定，出现问题的概率也会低很多。

![image-20230227193934377](/img/2023-02-27-minma.png)

#### 主题选择

由于 minma 实在是简单到过分，自己考虑选几个jekyll theme 来美化一下，遂有了以下的踩坑经历。

##### centarium

github：https://github.com/bencentra/centrarium

在google找到的比较符合自己品味的jekyll free theme，主题确实比较好看，但是在构建过程中，发现其使用到了jekyll-archives 的插件，然而 github pages 托管服务出于安全的考虑，只支持了很少一部分的 [jekyll插件](https://pages.github.com/versions/) ,该插件并不被支持，导致blog 在github pages 上显示总存在问题。

后来又通过google，找到在github pages 上绕过安全检测使用插件的[方法](http://ixti.net/software/2013/01/28/using-jekyll-plugins-on-github-pages.html)，在解决掉插件的问题后，部署完成后，自己感觉流程过于复杂，推送代码都要写个 rake task 不易维护，遂抛弃该方案。

![image-20230227195334526](/img/2023-02-27-centarium.png)

##### minimal-mistakes





### 致谢

