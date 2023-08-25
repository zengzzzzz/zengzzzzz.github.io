---
layout: post
title:  "为何存在uwsgi还要使用nginx"
date:   2021-05-19 19:47:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

&nbsp; &nbsp;nginx是对外的服务接口，外部浏览器通过url访问nginx，nginx接收到浏览器发送过来的http请求，将包解析分析url，如果是静态文件则直接访问用户给nginx配置的静态文件目录，直接返回用户请求的静态文件。  
&nbsp; &nbsp;若不是静态文件，动态请求的话，nginx将请求转发给uwsgi，uwsgi接收到请求后对包进行处理，处理成wsgi可接受的格式，并发送给wsgi，wsgi根据请求调用应用程序，将返回值交给wsgi，wsgi对返回值进行打包，并返回uwsgi-nginx-浏览器  
&nbsp; &nbsp;uwsgi完全可以单独组成和浏览器交互的流程，但是要考虑某些情况  
  
安全问题 nginx开放外部接口，uwsgi内网接口  
负载均衡 一个uwsgi不行，开启多个worker也不行，nginx做代理，可以代理多太uwsgi服务器  
静态文件处理效率 django uwsgi来处理静态文件是浪费，处理能力也不如nginx  
  
nginx 对静态文件的处理为何效率高  
  
open-file-cache 减少重复打开文件，sendfile系统减少内存复制，直接写到socket中  
epoll&nbsp;&nbsp;  
  
socket 发送文件流程  
  
打开文件  
文件数据读到内存  
内存数据写到socket  

