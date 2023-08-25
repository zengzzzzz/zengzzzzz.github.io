---
layout: post
title:  "python 中__new__与__init__区别"
date:   2021-06-01 19:31:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

二者都是python面向对象语言中的函数  
  
new在创建实例之前被调用，创建实例然后返回该实例对象，属于静态方法  
init是当实例对象创建完成后被调用的，设置对象的一些初始化值，属于实例方法  
先调用new，后调用init，new的返回值将传递给init方法，init给这个实例设置一些参数  
继承自object的新式类才能有new，new至少有一个参数cls，代表当前类  
new必须要有返回值，返回实例化出来的实例，可以return父类new出来的实例，在定义子类时没有重新定义new时，python默认调用该类的new方法构造该类的实例，如果该类的父类没有重写new，将一直追溯至object的new方法  
  
new的作用  
  
继承一些不可变的class时，比如int、str、tuple，只有重载它的new方法才能起到修改自定义的作用  

