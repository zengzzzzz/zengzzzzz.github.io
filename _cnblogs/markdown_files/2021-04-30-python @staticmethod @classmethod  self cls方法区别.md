---
layout: post
title:  "python @staticmethod @classmethod  self cls方法区别"
date:   2021-04-30 16:41:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

一直在用这些东西，但是又从来没有总结过，正好今日想起来就总结一下这些东西  
&nbsp;  
@staticmethod 静态方法，名义上归属类管理，不能使用类变量和实例变量，类的工具包放在函数前，不能访问类属性和实例属性，无须实例化，不传入cls，self  
  
&nbsp;  
&nbsp;@classmethod 函数不需要实例化，不需要self参数，第一个参数是表示自身类的cls参数，调用类的属性，方法，实例化对象  
  
&nbsp;  
@property 创建只读属性，将方法转换成相同名称的只读属性，与所定义的属性配合使用，防止修改  
&nbsp;  
&nbsp;  
self 为类的实例化对象，cls为类本身，@statticmethod 静态方法装饰器 @classmethod 类方法装饰器  
&nbsp;
