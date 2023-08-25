---
layout: post
title:  "mysql 中having 和 where 区别"
date:   2022-04-10 15:04:00
categories: 
tags:  zengzzzzz-blog
---

* content
{:toc}

在日常工作中有用到过where 与 having ，但是对其认知一直停留在比较简单的部分，今日特意整理一下其不同。  
&nbsp;  
声明角度：  
  
where 是约束声明，在查询数据库的结果返回之前对数据库中的查询条件进行约束，在结果返回之前起作用，在之后不能使用聚合函数  
having 是过滤声明，在查询数据库的结果返回之后进行过滤，即在结果返回之后起作用，在having之后可以使用聚合函数  
执行顺序： where &gt; sum min max avg group by&gt; having  
  
&nbsp;  
用法：  
  
where&nbsp; 在分组和聚集计算之前选取输入行， 而having 在分组和聚集之后选取分组行  
  
&nbsp;  
select sum(score) from student where sex = 'man' group by name having sum(score) &gt; 210  
&nbsp;
