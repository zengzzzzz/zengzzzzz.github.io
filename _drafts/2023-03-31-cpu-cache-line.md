---
layout: post
title:  "Module exponentiation"
date:   2023-03-23 20:10:00
categories: algorithm
tags: modular exponentiation
---

* content
{:toc}


## 多核环境下cache line 测试

前段时间在阅读 [uber/rat_limit](https://github.com/zengzzzzz/ratelimit/blob/main/limiter_atomic_int64.go) 代码时注意到其在定义struct时，会填入补充字段补至 64 byte来避免 cache line 的 false sharing，于是决定花些时间来探究一下其原理。

```golang
type atomicLimiter struct {
	state      unsafe.Pointer
	padding    [56]byte // cache line size - state pointer size = 64 -8 ; created to avoid false sharing
	perRequest time.Duration
	maxSlack   time.Duration
	clock      Clock
}
```

## CPU CACHE 简介

通常情况下，我们无法直接干预对缓存的操作，但可以根据缓存的特点对程序代码实施特定优化，从而更好的利用高速缓存。高速缓存策略会尽可能地将访问频繁的数据放入cache中，该过程是动态的，cache中的数据不会一直不变，目前一般机器的CPU cache 可分为一级、二级、三级缓存，较旧的机器可能只有二级缓存。该缓存和RAM在空间和效率上的关系如下：

         CPU
          |
          |  Fastest          Small
          | -----------------------  Level 1 Cache (L1)
          |                         |
          |  Faster               Small
          | -----------------------  Level 2 Cache (L2)
          |                         |
          |  Fast                 Small
          | -----------------------  Level 3 Cache (L3)
          |                         |
          |  Slowest              Large
          | -----------------------  Main Memory (RAM)

在lunix上可以使用lscpu来查看 cpu cache 的信息：

```
admin@al-hk-test-risk-app001:~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  2
Core(s) per socket:  4
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               85
Model name:          Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz
Stepping:            4
CPU MHz:             2499.996
BogoMIPS:            4999.99
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K # 一级数据缓存
L1i cache:           32K # 一级指令缓存
L2 cache:            1024K
L3 cache:            33792K
NUMA node0 CPU(s):   0-7
```

说明该机器为8核且存在三级缓存，由于不同处理器之间都具有自己的高速缓存，所以当俩个cpu cache中都存有数据a，就有可能需要进行同步数据。而cache直接进行同步数据的最小单元为cache行大小，每一行cache的大小都是64bytes，当cpu 被告知 cache 第一行的第一个byte 为脏数据时，cpu会将每一行都进行同步。

## 场景测试




