---
layout: post
title:  "Lunix kill 命令涉及到的一些问题"
date:   2022-04-14 11:05:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

Kill 命令本质上只是用来向进程发送一个信号，信号则是由于用户指定的  
eg : kill -15 pid  
&nbsp;  
0&nbsp; exit&nbsp; 程序退出时收到该信息  
1 hup 重新初始化进程  
2 int 表示结束进程， 但不是强制性的， ctrl + c  
3 quit 退出进程  
9 kill 强制结束进程  
15 term 正常结束进程 ，kill 默认
