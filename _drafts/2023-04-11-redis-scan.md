---
layout: post
title:  "Redis Scan - reverse binary iteration"
date:   2023-04-11 20:10:00
categories: algorithm redis
tags: redis iteration
---

* content
{:toc}

因之前阅读过redis中scan命令的merge，决定花些时间对其中的反向二进制迭代的原理进行解析。





### 摘要

本文旨在说明模幂运算的数学原理，并与快速幂进行对比分析。

#### 参考文章

https://www.cnblogs.com/thrillerz/p/4527510.html

https://github.com/redis/redis/pull/579

https://ifaceless.github.io/2019/12/17/implement-redis-dict-in-go/

https://github.com/huangz1990/annotated_redis_source.git

https://chromium.googlesource.com/external/github.com/gomodule/redigo/+/refs/heads/master/redis/scan.go

https://www.cnblogs.com/thrillerz/p/4527510.html

https://www.lixueduan.com/posts/redis/redis-scan/#4-%E5%B0%8F%E7%BB%93

