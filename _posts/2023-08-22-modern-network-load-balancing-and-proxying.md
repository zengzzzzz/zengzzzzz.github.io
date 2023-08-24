---
layout: post
title:  "modern network load balancing and proxying"
date:   2023-08-22 10:10:00
categories: network tech-article
tags:  network load-balancing proxying
---

* content
{:toc}

在过去的一个月里，我花了不少时间解决隔壁部门的网络负载均衡（LB）问题。认为这个经验不仅对于解决具体问题有帮助，还能够引发一些有关负载均衡和代理的深入思考（相关内容正在整理中）。正好，在这个过程中，我读到了一篇由Envoy的作者撰写的关于负载均衡和代理的文章。该文使我获益匪浅，因此，将该文章的内容进行梳理，既作为个人学习的笔记，也希望能与大家分享。





### 摘要
本文主要是对该[文章](https://blog.envoyproxy.io/introduction-to-modern-network-load-balancing-and-proxying-a57f6ff80236)内容采用思维导图的方式进行简单梳理，并提出一些自己的读后感，以供记录与分享。
### 思维导图
![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/envoy_lb/Introduction%20to%20modern%20network%20load%20balancing%20and%20proxying%20map.png)
### 读后感
这篇文章深入探讨了负载均衡（LB）的发展历史、现状以及未来发展方向。对我目前的阶段来说，它带来了很大的帮助，让我对LB的理解更加深入，同时也理解了一些LB的设计思想。今后应该常常回顾这篇文章，不断更新对负载均衡领域的认知。同时我也愈发认识到，文章作者对LB的理解远超过了自己短时间内能达到的水平。想要撰写出优质的技术文章，必须具备数倍于文章所涵盖内容的技术深度。

除技术方面外，我们应当逐步建立起自己的认知体系。提升认知，不仅停留在表面，还需要逐步形成属于自己的思考方法。这种认知的提升不仅仅限于技术领域，还贯穿于工作和生活的各个方面。通过持续学习、深入思考以及总结，逐步拓展自己的认知广度和深度，这也是持续成长的关键。

以最近读到的《送东阳马生序》中的一句话勉励自己，“勤且艰，其业有不精，德有不成者，非天质之卑，则心不若余之专耳，岂他人之过哉。” 
