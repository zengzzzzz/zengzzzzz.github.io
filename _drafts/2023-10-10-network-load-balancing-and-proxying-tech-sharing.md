---
layout: post
title:  "network load balancing and proxying sharing"
date:   2023-10-10 20:10:00
categories: network sharing
tags:  network load-balancing proxying
---

* content
{:toc}

最近公司内涉及到LB的调整较多，需要在公司内部分享一些关于LB的基础知识，这样做有两个主要目的。首先，希望通过分享来促进同事们在负载均衡（LB）领域的知识积累，使我们的团队更具专业知识。其次，借此机会，自己也打算梳理一下我自己在负载均衡领域的知识结构，以进一步提高专业素养。





### 摘要
这篇文章主要涵盖了现代网络负载均衡和代理的基本概念，侧重介绍了在负载均衡中的L4层和L7层之间的异同点，进行了详细的对比分析，并探讨了未来在这一领域的发展方向。

# 现代网络负载均衡及代理简介

## 负载均衡简介

### 负载均衡的定义

#### 维基

LB，是一种电子计算机技术，用来在多个计算机、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。 使用带有负载平衡的多个服务器组件，取代单一的组件，可以通过冗余提高可靠性。负载平衡服务通常是由专用软件和硬件来完成。 主要作用是将大量作业合理地分摊到多个操作单元上进行执行，用于解决互联网架构中的高并发和高可用的问题。

#### cloudfare

负载平衡是在两台或更多的计算机之间分配计算工作负荷的做法。在互联网上，负载平衡经常被用来在几个服务器之间分配网络流量。这能够减少每台服务器的压力，使服务器更有效率，加快性能，减少延迟。负载平衡对于大多数互联网应用的正常运行是必不可少的。

#### 腾讯云

负载均衡（Cloud Load Balancer，CLB）是对多台后端服务器进行流量分发的服务。负载均衡可以通过流量分发扩展应用系统对外的服务能力，通过消除单点故障提升应用系统的可用性。负载均衡服务通过设置虚拟服务地址（VIP），将位于**同一地域**的多台后端服务器资源虚拟成一个高性能、高可用的应用服务池。根据应用指定的方式，将来自客户端的网络请求分发到服务器池中。负载均衡服务会检查服务器池中后端服务器实例的健康状态，自动隔离异常状态的实例，从而解决了后端服务器的单点问题，同时提高了应用的整体服务能力。

#### AWS

负载均衡是在支持应用程序的资源池中平均分配网络流量的一种方法。现代应用程序必须同时处理数百万用户，并以快速、可靠的方式将正确的文本、视频、图像和其他数据返回给每个用户。为了处理如此高的流量，大多数应用程序都有许多资源服务器，它们之间包含很多重复数据。负载均衡器是位于用户与服务器组之间的设备，充当不可见的协调者，确保均等使用所有资源服务器。

#### 阿里云

阿里云负载均衡（Server Load Balancer，简称SLB）是云原生时代应用高可用的基本要素。通过将流量分发到不同的后端服务来扩展应用系统的服务吞吐能力，消除单点故障并提升应用系统的可用性。 阿里云SLB包含面向4层的网络型负载均衡NLB、面向7层的应用型负载均衡ALB和传统型负载均衡CLB，是阿里云官方云原生网关。

#### without LB

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/without_LB.png)

#### with LB

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/with_LB.png)

### 负载均衡器的作用

综上，在对LB的定义中可知，其前提是需要存在多个后端服务的，可用于提高服务吞吐量，处理海量流量，以达到服务高可用、高性能的目的。

#### 服务发现

对于LB本身的服务发现和负载均衡通常借助DNS，对后端服务的服务发现主要有client-side discovery、 server-side discovery、service mesh 等。常见的可以作为 service registry  有 zookeeper、Etcd、Consul等。

##### server-side discovery

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/server_side_discovery.png)

#### 健康检查

负载均衡器确定后端服务是否可用于服务流量的过程，一般分为被动、主动俩类，会针对不同的网络协议，做不同类型的健康检查。

##### 主动健康检查

LB定时主动向后端服务发起TCP连接或HTTP请求等，以此来衡量后端服务运行状况。

###### TCP 健康检查

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/TCP_health_check.png)

##### 被动健康检查

LB在向后端服务转发流量的同时记录后端服务的健康状态，达到累计失败次数，则转移后端流量。

#### 负载均衡

##### 静态负载均衡

静态负载均衡算法在分配负载时不考虑系统当前状态，并不关心后端服务是否充分利用。根据预先确定的计划分配工作负载，可快速设置。

###### 循环轮询

将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/robin_load.png)

###### 随机

通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。由概率统计理论可以得知，随着客户端调用服务端的次数增多，其实际效果越来越接近于平均分配调用量到后端的每一台服务器，也就是轮询的结果

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/random.png)

###### 加权轮询法

不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/weight_robin_load.png)

###### 加权随机

与加权轮询法一样，加权随机法也根据后端机器的配置，系统的负载分配不同的权重。不同的是，它是按照权重随机请求后端服务器，而非顺序。

###### IP/URL 哈希法

源地址哈希的思想是根据获取客户端的IP地址，通过哈希函数计算得到的一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/ip_hash.png)

##### 动态负载均衡

###### 最少连接法

最小连接数算法比较灵活和智能，由于后端服务器的配置不尽相同，对于请求的处理有快有慢，它是根据后端服务器当前的连接情况，动态地选取其中当前。

积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/least_conn.png)

###### 加权最小连接法

在最小连接数调度算法的基础上，根据服务器的不同处理能力，给每个服务器分配不同的权值，使其能够接受相应权值数的服务请求，是在最小连接数调度算法的基础上的改进。

###### 最短响应时间法

响应时间是服务器处理传入请求和发送响应所花费的总时间。最短响应时间法会将服务器响应时间与活动连接相结合，以确定最佳服务器。负载均衡器使用此算法来确保为所有用户提供更快的服务。

###### 基于资源的方法

自适应负载均衡算法，负载均衡器通过分析当前服务器负载来分配流量。称为代理的专用软件在每台服务器上运行，并计算服务器资源的使用情况，如其计算容量和内存。然后，负载均衡器将先检查代理是否有足够的可用资源，然后再将流量分配给该服务器。

###### P2C

P2C（Power of Two Choices）算法是一种基于随机化的负载均衡算法，它可以避免最劣选择和负载不均衡的情况。P2C算法的核心思想是：从所有可用节点中随机选择两个节点，然后根据这两个节点的负载情况选择一个负载较小的节点。这样做的好处在于，如果只随机选择一个节点，可能会选择到负载较高的节点，从而导致负载不均衡；而选择两个节点，则可以进行比较，从而避免羊群效应带来的最劣选择。

with weight

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/P2C_with_weight.png)

with P2C

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/P2C.png)

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/P2C_best.png)

#### 粘性会话

会话粘性（Session Affinity），也称为会话持久性（Session Persistence）或会话坚持（Session Stickiness），其中来自同一客户端的所有请求都被路由到相同的后端服务器。这样做的目的是确保在多个服务器之间保持用户的会话数据或状态的一致性。通常，会话粘性通过客户端的标识信息来实现，最常见的标识信息是客户端的 IP 地址或Cookie。

###### IP地址

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/session_ip.png)

###### Cookie

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/session_cookie.png)

#### TLS终止

在网络负载均衡器上执行 TLS 终止，可以将 TLS 流量的解密/加密工作从应用程序服务器分流至网络负载均衡器，从而优化后端应用程序服务器的性能，同时确保工作负载的安全。此外，网络负载均衡器会将客户端的源 IP 保存到后端应用程序，同时在负载均衡器上终止 TLS。可集中化部署 SSL 证书、选择性地向目标配置加密。可以灵活使用预定义的安全策略，以控制负载均衡器向客户端提供的密码和协议，实现强大可靠的应用程序安全状况。

#### 可观测性

网络本质上是不可靠的，负载均衡器通常负责导出统计数据、跟踪和日志，以确认找出问题所在，便于修复问题。负载均衡器提供丰富的输出，包括数字统计、分布式跟踪和可自定义的日志记录。负载均衡器必须做额外的工作才可产生它，数据的好处远远超过了相对较小的性能影响。

#### 安全性

在边缘部署拓扑中，负载均衡器通常实现各种安全功能，包括速率限制、身份验证和 防DDoS （例如，IP 地址标记和识别、缓存等）。

###### 阿里云DDoS原生防护

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/ali_defence.png)

#### 控制平面

即配置负载均衡器的系统，一般情况下包括控制平面UI、工作负载调度程序、服务发现、代理配置API。例如：Istio、Nelson、SmartStack 。

###### Istio  Control Panel

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/L4_L7_network/Istio_control_panel.png)

综上，LB主要用于提高后端服务应用程序可用性、应用程序可扩展性、应用程序安全、应用程序性能等。

### 负载均衡拓扑类型

#### Middle proxy

![image-20231015160026547](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015160026547.png)

中间代理模式为最常见的代理模式，其优点是简单，用户一般只需要通过 DNS 连接到 LB。其缺点是，这种模式下负载均衡器（包含集群）是单点的（single point of failure），且横向扩展有瓶颈。

中间代理很多情况下都是一个黑盒子，给运维带来很多困难。例如发生故障的时候，很难判 断问题是出在客户端，中间代理，还是后端。常见的有 HAProxy、NGINX、Envoy等。

#### Edge proxy

![image-20231015160034771](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015160034771.png)

边缘代理拓扑只是中间代理拓扑的一个变种， 这种情况下负载均衡器可以从公网直接访问。该场景下， 负载均衡器通常还要提供额外的API 网关功能， 例如 TLS termination、限速、鉴权，以及复杂的流量路由等等。

中间代理拓扑的优缺点对边缘代理也是适用的。对于面向公网的分布式系 统，部署边缘代理通常是无法避免的，客户端一般通过 DNS 访问系统。 此外，所有来自公网的流量都通过唯一的网关进入系统在安全角度方面而言是比较好的。常见的阿里云NLB、SLB等。

#### Embedded client library

![image-20231015160043547](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015160043547.png)

客户端内嵌库，为解决中间代理等固有的单点、扩展等问题，可以将负载均衡 器已函数库的形式内嵌到客户端。该方式最大的有点在于将 LB 的全部功能下放到每个客户端，从而完全避免了单点和扩展问题，其缺点在于用的每种语言实现相应的库，在大型服务架构，很可能导致生产集群中同时运行多个版本的客户端， 增加运维成本。常见的有gRPC、Finagle、 Hystrix等。

#### Sidecar proxy

![image-20231015160100377](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015160100377.png)

sidecar 即为客户端内嵌库拓扑的一个变种，也被称为 service mesh。与客户端内嵌库不同点在于，sidecar是将流量导到另一个进程，牺牲一点（延迟）性能，实现客户端内嵌库模式的所有好处，而无任何语言绑定。常见的有 HAProxy、NGINX、Envoy、Linkerd等。

## L4（会话/连接）负载均衡

### 基本工作原理

传统的L4 TCP 负载均衡器，客户端建立一个 TCP 连接到 LB。LB 终止这个连接，选择一个后端，然后建立一个新的 TCP 连接到后端。L4 负载均衡器只工作在 L4 TCP/UDP connection/session 。因此，LB 在双向来回转发字节，保证属于同一 session 的请求可以到同一后端。L4 LB 不会感知其转发请求所属应用的任何细节，可以是任何其他应用层协议。L4 在仅对整体流量负责，不会对具体的请求负责，当使用keep-alive时，可能会较多的存在具体的请求分布不均匀。

![image-20231015162638940](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015162638940.png)

### L4 相关内容

###### TCP/UDP termination

![image-20231015185003781](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015185003781.png)

该模式中，会使用两个独立的 TCP 连接：一个用于客户端和负载均衡器之间，一个用于负载均衡器和后端之间。

该模式实现相对简单，距离client 距离越近，则性能越好。

###### TCP/UDP passthrough

![image-20231015185322512](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015185322512.png)

在该模式中，TCP连接不会被LB terminate，而是在建立连接跟踪和网络地址转换（NAT）之后直接转发给选中的后端。例如，我们假设客户端正在和负载均衡器 A通信，选中的后端是 B。当客户端的 TCP 包到达负载均衡器时，负载均衡器会将包的目的 IP/port 换成 B，以及将源 IP/port 换成负载均衡器 自己的 IP/port。当应答包回来的时候，负载均衡器再做相反的转换。该种模式主要的优点在于无须缓存任何TCP连接窗口，性能及资源消耗较小；允许后端进行自主拥塞控制，该模式不参与拥塞控制，可使后端服务自行选择。

###### DSR

![image-20231015190242564](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015190242564.png)

该模式基于passthrough，只允许进来的流量 /请求（ingress/request）经过 LB，而出去的流量/响应（egress/response）直接 从服务器返回到客户端。设计该模式的主要原因在于，在一些场景中，响应的流量远大于请求流量，采用DSR，可以节约LB的成本，提高可靠性，但仍然存在一些与passthrough不一致的方面，LB无法知道TCP连接的完整状态；LB 可能不采用NAT 而是GRE，将IP包给到后端服务，后端可以直接将应答包返回客户端；后端可能需要参与负载均衡过程，配置GRE。

###### HA pair

![image-20231015191536121](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015191536121.png)

为避免单个L4挂掉导致应用不可用，LB 一般以HA pair 方式部署。典型的HA 负载均衡设置包括： 一对 HA 边缘路由器提供若干虚拟 IP（virtual IP，VIP），并通过 BGP (Border Gateway Protocol) 协议宣布 VIP。主边缘路由器（primary）的 BGP 权重比备边缘路由器（backup）的高， 在正常情况下处理所有流量。primary L4LB 向边缘路由器宣告它的权重比 backup LB 大，因此正常情况下它 处理所有流量。两个边缘路由器和两个负载均衡器都是交叉连接的。如果一个边缘路由器或一 个负载均衡器挂了，或者由于某种原因之前声明的 BGP 权重收回了， backup 马上可以接受所有流量。在该种模式下资源利用率较低，主备仍可能同时挂掉。

###### Consistent Hashing

![image-20231015194305957](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015194305957.png)

该模式为大规模并行L4 负载均衡系统，基于集群化与一致性哈希实现容错与可扩展，其工作原理如下：

N 个边缘路由器以相同的 BGP 权重通告所有 VIP。通过 ECMP（Equal-cost, Multi-path routing，一种通过一致性哈 希将包分发到一组权重相同的网络设备的方式）保证每个 flow 的所有包都会到达同一个边缘路由器。边缘路由器通常并不关心每个包要 发往哪里，但一般希望同一 flow 的所有包都以相同路径经过各个设备，这样可以 避免乱序代理的性能下降。N个 L4LB 以相同的 BGP 权重向所有的边缘路由器通告所有的 VIP。仍然使用 ECMP，边缘路由器会为相同 flow 的包选择相同的 LB。每个 L4 LB 实例会做部分连接跟踪工作，然后使用一致性哈希为每个 flow 选择 一个后端。再通过 GRE 封装将包从 LB 发送到后端。然后使用 DSR 将应答包从后端直接发送到边缘路由器，最后到客户端。

该种模式中的边缘路由器和负载均衡实例均可按需添加，每层都用到了ECMP，当新实例加入时，ECMP 可最大程度减少受影响的flow数量；预留足够容错前提下，系统资源利用率可以达到很高。

常见的支持L4的一些产品：NGINX、HAProxy、KeepAlived、Traefik、Caddy

## L7 （应用）负载均衡

### 基本工作原理

![image-20231015200414847](C:\Users\10269\AppData\Roaming\Typora\typora-user-images\image-20231015200414847.png)

在该模式下，客户端与 LB 只建立一个 HTTP /2 TCP 连接。LB 接下来和两个后端建立连接。当客户端向 LB 发送两个 HTTP/2 流（streams ）时，stream 1 会被发送到后端 1，而 stream 2 会被发送到后端 2。因此，不同客户 端的请求数量差异巨大，这些请求也可以被高效地、平衡地分发到后端。

### L7 相关内容

#### service mesh

![image-20231016143317235](/Users/zengzh/Library/Application Support/typora-user-images/image-20231016143317235.png)

LB在云原生应用程序中是动态的，由于所有的动态部分，该应用程序可能具有不同的性能。服务网格中的负载均衡器在向各个实例发送请求之前需要考虑它们的运行状况。它可以阻止或路由流量绕过不健康的实例，帮助避免紧急情况并提供更可靠的服务。

负载均衡器可以主动轮询服务网格的服务发现部分，检查运行状况良好的实例，也可以被动响应失败的请求并仅根据性能切断实例的流量。

除此之外，服务网格中的负载平衡使用算法来决定如何在网络上路由流量。过去，路由很简单，使用循环或随机路由等方法。借助现代服务网格，负载平衡算法现在会考虑后端实例上的延迟和可变负载。例如 Istio、envoy、LINKERD、CONDUIT等。

#### API Gateway

![image-20231016144409301](/Users/zengzh/Library/Application Support/typora-user-images/image-20231016144409301.png)

API 网关位于每个 API 请求的执行路径上，是一个数据平面，用于接收来自客户端的请求，并可以在最终将这些请求反向代理到底层 API 之前强制执行流量和用户策略。在将请求代理回原始客户端之前，它还可以（而且很可能会）对从底层 API 收到的响应执行策略。

API 网关可以具有内置控制平面来管理和配置数据平面的功能，也可将数据平面和控制平面全部捆绑到同一进程中。

API 网关部署在其自己的实例（其自己的 VM、主机或 Pod）中，与客户端和 API 分开。例如envoy、NGINX、APISIX、KONG等。

#### 可扩展性

![image-20231016145713518](/Users/zengzh/Library/Application Support/typora-user-images/image-20231016145713518.png)

采用集群化部署LB，确保负载均衡器自身不会成为单点故障点。可基于负载均衡器实际负载，对其进行自动伸缩容，维持稳定的负载均衡集群。

#### 分布式追踪

![image-20231016151534346](/Users/zengzh/Library/Application Support/typora-user-images/image-20231016151534346.png)

负载均衡器可通过插件的方式集成分布式链路追踪框架，实现分布式链路追踪功能。例如：Zipkin、OpenTracing 等。

#### 协议支持

现代 L7 负载均衡器正在显式添加对更多协议的支持，对应用层协议了解的越多，则可处理更多更复杂的事情，包括观测输出、高级负载均衡和路由等等。现在大部分的L7都会显式支持解析和路由HTTP/1、HTTP/2、HTTPS、websocket、gRPC、Redis、MongoDB、MySQL、Kafaka等。

## 4层负载均衡与7层负载均衡的对比

现在不少应用对L4 L7负载均衡均提供了技术支持，例如NGINX、Envoy、Traefik 等。但是为方便我们在日常工作中的技术选型，结合上述内容，仍对L4 L7负载均衡在以下方面做出对比分析，并描述在实际使用场景中的应用。

### 对比分析

- OSI层级
  - 4层负载均衡：工作在OSI模型的传输层（第四层），主要基于IP地址和端口号进行负载均衡，通常实施在网络层。
  - 7层负载均衡：工作在OSI模型的应用层（第七层），深入到应用层协议（如HTTP、HTTPS）的内容中，根据请求的内容，如URL、HTTP标头等进行负载均衡。
- 负载分配粒度
  - 4层负载均衡：分配请求基于传输层协议（通常是TCP或UDP）的端口号，不了解请求的具体内容。通常基于源IP和目标IP地址以及端口号进行负载均衡。
  - 7层负载均衡：分配请求基于应用层协议和内容，可以识别不同的HTTP请求，并做出基于请求的智能决策。除了传输层负载均衡算法外，还可以基于应用层数据，如HTTP请求头、URL路径等，进行更高级的路由和负载均衡。
- 应用场景
  - 4层负载均衡：适用于大多数基于TCP/UDP的应用，一般用于网络边缘，如数据库、DNS等。
  - 7层负载均衡：适用于Web应用，需要深度内容分发和应用层路由的应用，一般使用在service to service 场景中，如网站、API服务等。

### 实际使用场景

在 service-to-service 通信中 L7 负载均衡是可以完全取代 L4 负载均衡，但 L4 负载均衡在边缘仍非常重要，大多数现代大型分布式架构都是在公网流量入口使用 L4/L7 两级负载均衡架构。 其原因主要在于：

- L7 LB 承担的更多工作是复杂的分析、变换、以及应用流量路由，一般情况下处理原始流量的能力比 L4 负载均衡器要差。则L4 LB 更适合处理特定类型的攻击，例如 SYN 泛洪、通用包泛洪攻击等。
- L7 LB 功能更多更复杂且部署的更多更频繁，bug 也比 L4 LB 多。在 L7 之前加一层 L4LB，可以在调整 L7 部署的时候，对其做健康检查和流量排除。

## 全局负载均衡及负载均衡的未来

### 全局负载均衡

![image-20231016113729825](/Users/zengzh/Library/Application Support/typora-user-images/image-20231016113729825.png)

每个 sidecar 同时和位于三个 zone 的后端通信，90% 的流量到了 zone C，而 zone A 和 B 各只有 5%；sidecar 和后端都定期向全局负载均衡器汇报状态，全局负载均衡器可以基于 延迟、代价、负载、当前失败率等参数做出决策，全局负载均衡器定期配置每个 sidecar 的路由信息。

### 负载均衡的现在与未来

- 云原生架构：云原生应用程序采用微服务架构，未来的负载均衡将与容器编排平台（如Kubernetes）集成，以支持动态负载均衡、服务发现和自动扩展。
- 安全性增强：安全性将继续是一个关键关注点。负载均衡设备将提供更多的安全功能，如DDoS攻击检测和防护、Web应用程序防火墙（WAF）、SSL终结等，以保护应用程序免受各种网络威胁。
- 边缘计算支持：随着边缘计算的兴起，负载均衡将更频繁地出现在边缘节点上，以将应用程序和内容更接近用户。这将需要负载均衡设备在边缘节点上提供高性能和低延迟的负载均衡服务。
- 资源利用效率：负载均衡将变得更加高效，以降低成本。
- API驱动的配置：未来的负载均衡设备将提供更多的API和自动化工具，使开发人员能够以编程方式配置和管理负载均衡策略，从而实现更快速的应用部署和更新。
- 全局负载均衡：数据平面与控制平面的分离。

## 参考文章

https://en.wikipedia.org/wiki/Load_balancing_(computing)

https://blog.envoyproxy.io/introduction-to-modern-network-load-balancing-and-proxying-a57f6ff80236

https://aws.amazon.com/cn/what-is/load-balancing/

https://www.aliyun.com/product/slb

https://cloud.tencent.com/document/product/214/524

https://www.cloudflare.com/zh-cn/learning/performance/what-is-load-balancing/

https://brooker.co.za/blog/2012/01/17/two-random.html

https://blog.envoyproxy.io/examining-load-balancing-algorithms-with-envoy-1be643ea121c

https://bbs.huaweicloud.com/blogs/193876

https://cloud.tencent.com/document/product/214/6097

https://mikechen.cc/14875.html

https://github.com/zeromicro/go-zero

https://github.com/envoyproxy/envoy

https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/

https://kemptechnologies.com/load-balancer/load-balancing-algorithms-techniques

https://dubbo.apache.org/zh-cn/overview/reference/proposals/heuristic-flow-control/

https://zhuanlan.zhihu.com/p/401118943

https://www.nginx.com/blog/nginx-power-of-two-choices-load-balancing-algorithm/

https://aws.amazon.com/blogs/aws/new-tls-termination-for-network-load-balancers/

https://help.aliyun.com/zh/slb/classic-load-balancer/user-guide/anti-ddos-origin-basic

https://blog.envoyproxy.io/service-mesh-data-plane-vs-control-plane-2774e720f7fc

https://istio.io/

https://chat.openai.com/

https://getnelson.io/

https://github.com/airbnb/synapse

https://github.com/Netflix/Hystrix

https://developer.aliyun.com/ask/241594

https://datatracker.ietf.org/doc/html/rfc2784

https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html

https://github.com/yyyar/gobetween

https://www.nginx.com/resources/glossary/layer-4-load-balancing/

https://github.com/acassen/keepalived

https://www.hashicorp.com/blog/layer-7-observability-with-consul-service-mesh

https://levelup.gitconnected.com/l4-vs-l7-load-balancing-d2012e271f56

https://www.cloudbees.com/blog/an-overview-of-the-service-mesh-and-its-tooling-options

https://konghq.com/blog/enterprise/the-difference-between-api-gateways-and-service-mesh

https://www.qingcloud.com/products/loadbalancer/

https://www.alibabacloud.com/help/en/tracing-analysis/latest/use-zipkin-to-perform-tracing-analysis-on-nginx