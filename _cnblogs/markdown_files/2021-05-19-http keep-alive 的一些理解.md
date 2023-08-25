---
layout: post
title:  "http keep-alive 的一些理解"
date:   2021-05-19 19:26:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

&nbsp; &nbsp;在日常的工作中涉及到了关于检测用户是否在线的功能需求，在方案设计过程中考虑过使用http-keepalive的方式，由于我们获取到的keepalive是经过nginx转发的keep-alive，所以最后选择了实时更新数据库的方案，借此机会总结一下关于http-alive的一些问题。  
http keepalive 介绍  
&nbsp; &nbsp;在http早期，每个http请求都要求打开一个tcp socket连接，并且使用一次之后就断开这个tcp连接。使用keep-alive，即在一次TCP连接中可以持续发送多份数据而不会断开连接。通过使用keep-alive机制，可以减少tcp连接建立次数，也意味着可以减少TIME_WAIT状态连接，以此提高性能和提高httpd服务器的吞吐率(更少的tcp连接意味着更少的系统内核调用,socket的accept()和close()调用)。  
&nbsp; &nbsp;长时间的tcp连接容易导致系统资源无效占用。配置不当的keep-alive，有时比重复利用连接带来的损失还更大。所以，正确地设置keep-alive timeout时间非常重要。  
  
&nbsp; &nbsp;Httpd守护进程，一般都提供了keep-alive timeout时间设置参数，比如nginx的keepalive_timeout，这个keepalive_timout时间值意味着：一个http产生的tcp连接在传送完最后一个响应后，还需要hold住keepalive_timeout秒后，才开始关闭这个连接。当httpd守护进程发送完一个响应后，理应马上主动关闭相应的tcp连接，设置 keepalive_timeout后，如果守护进程在这个等待的时间里，一直没有收到浏览发过来http请求，则关闭这个http连接。   
&nbsp;  
nginx keepalive  
&nbsp;  
&nbsp; nginx的keepalive是从最后一个数据包开始计算的  
&nbsp;  
http keepalive 实践  
&nbsp;  
&nbsp; &nbsp;nginx代理与上游服务器upstream之间的连接默认是关闭了长连接  
  
  
keepalive：4 每个nginx worker最多可保留的长连接，超过这个数量则会根据LRU算法关闭  
keep-alive-timeout：60 最大空闲长连接  
keepalive-requests：100 每个长连接最多能处理的请求次数  
  
&nbsp; &nbsp;location 配置  
  
connection close删掉  
http-version 1.1  
  
uwsgi 额外配置keepalive  
  
uwsgi 支持keep-alive 若nginx通过http-pass；但是通过uwsgi-pass则不行  
  
requests nginx keepalive  
  
requests默认不支持keepalive，需要用requests.session 使用keepalive  
  
  
&nbsp;  
&nbsp;  

