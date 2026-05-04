---
title: 静态IP 动态IP 浮动IP 虚拟IP 区别是什么？
date: 2021-11-16 18:17:21
tags:
	
	- 计算机网络
---



static ip 就是固定分配的 ip，需要手工管理，非常麻烦。为了减少麻烦，人们发明了 dhcp 协议，来自动为电脑分配 ip，这就是 dynamic ip。floating ip 跟 dynamic ip 有点像，参考各公有云厂商的弹性 ip。

但不论 static ip、dynamic ip 还是 floating ip，一个 ip 只能分配给一台电脑。

在有些情况（比如高可用场景）下我们需要多台电脑共用一个 ip，也就是说一个 ip 「属于」多台电脑。那怎么实现呢？是给两台电脑设置同一个 ip 吗？显然不是，因为为产生 ip 冲突。这就需要 virtual ip。

比如我们有两台服务器 AAA 和 BBB，它们的 IP 分别是 10.0.0.1 和 10.0.0.2。它们功能相同，提供相同的服务。

理论上大家可以直接能过 10.0.0.1 或者 10.0.0.2 来访问 AAA 或 BBB 的服务。但如果某一台机器宕机，就没法访问了。要解决这个问题就需要 virtual ip。

![](https://pic3.zhimg.com/v2-c78f0bd0d9d71a6add9114fa026a1223_r.jpg?source=1940ef5c)

首先，我们从 AAA 和 BBB 中选一个作主，另一个作备。然后要求它们互相探测，确保对方都在线。然后给 AAA 和 BBB 同时「分配」一个 virtual ip 10.0.0.100。其他主机需要通过 10.0.0.100 来访问 AAA 或 BBB 提供的服务。

一般来说，其他主机要访问 10.0.0.100 需要通过 ARP 获取对应的 MAC 地址。如果 AAA 和 BBB 同时应答 ARP 请求，就会产生冲突。因为 10.0.0.100 是 virtual ip，所以，只有主服务器 AAA 才能应答。BBB 收到 ARP 请求后发现 AAA 还活着，就自动闭嘴。

![](https://pic1.zhimg.com/v2-06fd6ab924a50e4d766142bcdd9c7ac3_r.jpg?source=1940ef5c)

之后所有访问 10.0.0.100 这个 virtual ip 的请求都会发到 AAA。

如果 AAA 出现故障呢？这个时候其他主机发现 10.0.0.100 不通了，于是发出新的 ARP 请求

![](https://pica.zhimg.com/v2-42d63449da83eee4588e1d677004f25a_r.jpg?source=1940ef5c)

同时，BBB 也探测不到 AAA，它知道自己的高光时刻来到了，于是 BBB 大声响应 ARP 请求说「向我开炮」。

![](https://pic3.zhimg.com/v2-93db9c7a1e350368a59e32da19ebfec8_r.jpg?source=1940ef5c)

总结下来，virtual ip 就是多主机设置相同 ip，但只有一台主机可以在特定条件下响应 arp 请求。

