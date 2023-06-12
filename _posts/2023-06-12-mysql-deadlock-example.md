---
layout: post
title:  "mysql deadlock example"
date:   2023-06-12 17:10:00
categories: mysql work-example
tags: mysql innodb deadlock
---

* content
{:toc}


在2023-06-05出现线上数据库死锁问题，在我们的系统中死锁问题并不是很常见，所以本次死锁问题虽较为简单易解，仍需要简单记录，以供学习反思。







### 摘要
本文旨在说明2023-06-05日线上出现的数据库死锁问题，并提供合理的解决方案。
### 背景
数据库版本为 mysql 5.7 innodb
数据库死锁日志：

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/deadlock_example/deadlock_log.png)

本来以为是比较少见的gap lock 带来的死锁问题，找DBA看过死锁日志（只保留最后一次死锁记录）之后，应该就是比较常见的事务之间对资源顺序交替访问导致的死锁。
### 问题分析
#### 问题定位      
 根据线上告警日志可定位到是 A 查询定时任务在更新数据表时发生了死锁，阅读其发生死锁部分的代码，发现其在update 时均指定索引，排除是该处更新任务并发执行时导致的死锁。之后找DBA查找死锁日志之后，可以比较明确的看出是前段时间上线的B优先级过期定时任务 和 之前的A查询外发定时任务引发的死锁，只是数据库选择了代价更小的事务进行了回滚，服务错误日志会报在A 个人查询处。
#### 问题原因
A查询外发定时任务相关事务(以下称为事务A)：

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/deadlock_example/trans_A.png)

B 查询优先级过期相关事务（以下称为事务B）：

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/deadlock_example/trans_B.png)

从上述俩副图中可以明显看出事务A与事务B 对 r_slik_to_send 及 r_slik_credit_record 的更新顺序是相反的，且事务B为批量更新操作，则会存在较小的概率发生，事务A、事务B 对 r_slik-to_send 、r_slik_credit_record 表中同一条数据更新时产生互相等待的死锁问题。
死锁发生条件示意图：

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/deadlock_example/deadlock_situation.png)

在该场景下事务A、B 都会对r_slik_credit_record 中的 a 数据以及r_slik_to_send中的b数据进行更新，当事务A更新a数据之后，需要获取b数据的锁，等待更新b数据，而事务B则刚好相反，则便会出现事务A等待b数据锁，事务B等待数据a锁，即死锁。
### 解决方案
#### 方案一：拆分大事务
简单粗暴的做法，将事务A、B均拆分为小事务，则不会出现该问题，但是应该会违背代码作者使用事务的用意。
#### 方案二：拆分批量更新
将事务B中的批量更新拆分开，该方法会降低出现死锁的概率，单条数据更新时，出现同时更新俩个表中的相同数据概率还是要比原来低的，但是并没有从根本上解决问题，反而拆分会更复杂。
#### 方案三：调整更新顺序
调整事务A/B中的sql执行顺序，保持一致的更新顺序即可，避免死锁，该方案简单有效。
综上，选择方案三以解决死锁问题。在日常编码过程中，进行大事务处理时，应该更加谨慎一些，考虑范围要更加全面一些，避免大事务可能会带来的一些问题。
### 参考文章
https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html