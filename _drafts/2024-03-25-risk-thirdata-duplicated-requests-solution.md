---
layout: post
title:  "risk thirdata duplicated requests solution"
date:   2024-03-25 19:10:00
categories: system-design  tech-article
tags:  system-design architecture
---

* content
{:toc}

2024年春节后由于日常的事情较多，攒了比较久的blog都没有更新。恰逢在春节期间发现部门内部的一些对外请求服务，由于之前同事在设计时并未充分考虑并发问题而导致出现大量重复请求的情况，遂对其进行了优化，解决了重复请求的问题，也在此基础上整理并分享了一些避免对外重复请求的方案。




### 摘要
这篇文章主要介绍风控中台对外部服务的重复请求问题的解决方案，因其主要用作内部分享，所以其中涉及到的大多是一些内部具体实现的实例，对于外部的读者会不太友好，之后应该会进行更加通用化的整理。



# 并发问题处理
## 目前面临的并发问题
我们目前主要的三方服务在线上采用在多服务器上多进程的方式进行部署，当线上出现并发请求时，会导致我们线上出现重复无效请求，造成资源浪费。我们目前主要的并发问题存在以下俩种情况：
### 同步请求并发问题（以pefindo为例）
该种并发情况通常会在什么情况下出现
  1. 对同一外部请求服务，同一单，在第一次请求未结束前，便开始第二次请求
  2. 对同一外部请求服务，不同单，不同的flow，对同一个 ic no 进行并发请求
即可概况为在同一个ic no 的外发请求期间，并未写入data表时，陆续进入的对同一 ic no的请求均会因查询不到缓存而外发。
![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/pefindo_requests.png)
### 异步请求并发问题（以MetaMap、BAP为例）
对于异步请求并发处理中，主要存在以下俩种情况：
  1. 类似metamap 类型的异步请求服务，流程较为简单，不存在定时任务的情况，可直接请求外部，其可能因并发导致的重复请求问题与在同步并发请求中的描述是一致的。
  ![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/metamap_requests.png)
  2. 类似bap这种批量请求的异步服务，query接口仅为发送数据表中插入需要的请求数据，真实的请求外发流程则依赖其他任务，其存在的并发问题除上述同步并发请求中的描述中的并发问题（bap query 接口中）之外，仍存在每次批量外发任务启动时，可能会捞取重复的数据（下图中的get search data）进行外发，尤其是当我们使用landsat数据平台进行调度时，由于landsat平台调度的问题，会导致启动任务时的间隔出现不稳定的情况，导致上次任务未结束，下次任务已开启，外发重复数据。
  ![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/bap_requests.png)
## 为什么需要处理并发问题
处理并发问题并不是我们的目的，我们的目的是处理重复请求的问题，只是并发问题会给我们的服务带来重复的外发请求，所以我们需要处理并发问题。处理并发问题的原因主要存在以下几点：
  1. 重复请求会造成外部请求资源浪费，导致无效请求，造成重复收费的问题（最主要的原因）
  2. 对于某些外部服务其本身会限制我们的流量，当我们出现重复请求时，会导致一些正常的请求无法外发，使系统变得低效
  3. 出现大量的无效外发请求时，有可能会打挂对方服务，出现对我们请求的熔断，导致我们系统无法正常外发请求
## BAP是怎么处理的
在bap对于并发导致的重复数据请求处理流程中着重介绍关于landsat触发批量外发请求的这部分内容
### BAP 处理重复外发简要流程
  1. 通过query接口插入数据使用md5保证单个人的数据在 record status表中是不重复的。
  2. 在获取数据后采用对record 表中某条记录的状态流转作为锁，锁定该条记录已被发送，不再被其他轮次的任务获取该条数据外发，即一条数据只能被一个任务外发。
  3. 在该条数据出现外发失败等意外情况时，回滚该数据的状态，保证其可被下一轮次的任务发送。
### BAP 如何保证某条已外发的数据不被其他轮次的任务获取
  1. get serach data 时获取数据过滤条件为 status = 200
  2. 在外发前更新数据 set status = 2001 where status = 200，并发更新时，只会存在一个任务更新成功，另一个更新失败
  3. 更新成功的任务外发该数据 status = 201
### BAP 处理数据时为何不锁定任务，而是锁定数据
  1. 锁定任务即在下次调用时，若上次调用未结束，则不启动本次任务，在该场景下，可处理landsat俩次调用之间的问题，但会影响外发请求效率，遂选择锁数据
  ![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/bap_solution.png)
## dkp是怎么处理的
### dkp 处理重复外发简要流程
  1. 在请求进入时查询记录，若无记录则插入lock，请求外部
  2. 当存在结果时，直接返回结果，不外发请求
  3. 当不存在结果时，检查是否存在lock （时间过期判断），存在 lock 则返回限制
  4. lock 过期则更新lock结果，请求外部
### dkp 如何保证某个处于处理中的单不会被其他请求处理
  1. 在 insert lock 时 采用 uniq risk id 唯一键，同一uniq_risk_id 下的单 只会有一单插入 `uniq_risk_id` (`risk_id`,`source_id`,`scene_id`,`channel_id`,`business_type`,`ic_no`),
  2. 在 update lock 时，用 uniq risk id and lock = 0 过滤，保证同一 uniq_risk_id 下的单，只会有一单被update
  ![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/dkp_solution.png)
# 单维度限制兜底策略
### 为什么需要做单维度限制外发兜底
在上一单未结束情况下，我们应该限制同一单的数据不重复外发，避免重复请求导致的外部请求资源浪费，造成重复收费。
### 应该怎么处理单维度兜底
对于单维度兜底策略我们需要处理的情况，包括但不限于相同单出现并发的情况，主要是针对相同数据在上一次外发未完成的情况下，开启第二次或者n次外发时的情况。我们考虑的方案主要提供以下俩种：

  | 方案           | 具体内容                                                     | 优点                                                         | 缺点                                                         |
  | :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | 类似 dkp info  | 采用  `uniq_risk_id` (`risk_id`,`source_id`,`scene_id`,`channel_id`,`business_type`,`ic_no`) 定义单维度限制重复请求，每个单只会外发一次 | 单维度定义清晰简洁单维度定义变更概率较小，一般不需要修改条件判断条件较为标准，且存在dkp作为参考，可复制性高 | 无法处理不同单中出现相同数据的情况，造成相同数据请求外部资源浪费 |
  | 类似 bap query | 采用 unique_key,uid+MD5(uid, business_type, first_name, middle_name, last_name, birthdate)[-8:] 作为唯一键限制重复请求，每条数据只会外发一次 | 可避免不同单中相同数据发送外部请求的情况，过滤重复单的范围比dkp 方案更好 | 判断条件较复杂，且容易发生变更，可复制性不高当修改条件时，我们发布又是采用单进程，单机器的模式发布，会导致在发版期间内暂时失效 |

### 为什么在存在人维度缓存的情况下仍需要做单维度兜底
  1. 在并发情况下，简单的人维度缓存无法对多次相同数据请求进行限制，可能导致多次外发。单维度主要是作为一个兜底限制，避免特殊情况或者bug导致的重复外发。
  2. 有些业务场景不适合用人维度做并发限制。例如，对于dkp info 这种不使用缓存的外部请求服务，我们是无法利用缓存命中去处理过滤一部分重复请求的。
  3. 单维度兜底和人维度缓存并不矛盾，两者的目标不同，技术上可以考虑融合实现，确保控制好并发风险。
# 我们应该怎么处理（处理规范）
## 同步/异步单次请求 处理方式
对于同步或异步单次请求的服务，我们只需要保证在发送时为数据添加互斥锁，保证符合我们互斥条件的数据，只会有一条外发即可。
### 单维度锁且人维度锁简要限制外发流程图
![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/risk_person_lock.png)
### 单/人 维度锁流程图（包含cache的常规服务）
![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/risk_lock.png)
### 单维度锁且人维度锁限制外发流程图
![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/risk_person_lock_detail.png)
## 异步批量 处理方式
在异步批量处理重复数据的问题中，我们首先会保证在query接口中插入to send 表中的数据均是唯一的（通过同步 异步单次请求中的逻辑保证），我们只需要处理不同轮次的任务不要获取到重复数据即可。则在异步批量处理中，不需要关心单维度或者人维度，已经在插入数据时处理完成，仅需对需要发送的数据添加互斥锁即可。
### 异步批量处理流程图
![img](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/risk_thirdata_duplicated_request_solution/async_data_lock.png)
## 可能存在的问题
### 程序异常中断应该怎么处理
当程序异常中断时，我们的所设置的异常情况，状态回滚、锁释放均会失效，导致锁一直存在，对于该种情况，我们可以采用时间过期的策略等，但目前该类异常数据在线上存在较少，可暂时采用数据监控，配合人工修复异常数据即可。
### 当锁的条件发生变更时，在发版过程中重复请求数据怎么处理
该种情况主要指的是，由于我们本身服务的部署及发版方式，会导致在发版时，线上会短暂存在俩个版本的代码同时运行，在这种情况下，当我们对锁的条件发生改变时（类似缓存条件变更等），是会造成重复数据请求的，在该种情况下，我们如果具有单维度限制（正常情况是不会发生条件变更的）的兜底策略是可以避免很大一部分无效请求的。
# 总结
本文仅以风控系统部分三方服务为例说明我们现有系统存在的部分服务外发重复数据的问题及为我们如何处理该类问题提供了一些解决方案以供参考，但是在实际工作的过程中，对于具体的服务我们还是需要进行具体分析的。

