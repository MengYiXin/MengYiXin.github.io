---
title: 关于Docker的缺点
date: 2023-2-02 20:05:21
tags:
	
	- 桌面云
---




使用 docker 时候，我惊叹发现 docker 真的是个宝藏，可以秒级开启应用，不用特别繁琐的配置什么配置，可以减轻很多这种重复性的劳动。最近我在学习 docker，然后我就在想它可不可以虚拟化 Windows 的桌面系统呢？

通过查找资料，我发现 docker 不适合虚拟化桌面，它比较适合 linux 族，Unix 族以及应用程序的虚拟化。

逻辑是这样的，在操作系统的概念里面，有一个内核，docker 是基于 Linux 内核，它不适用于 Windows 虚拟化，Windows 操作系统有另外一个虚拟技术 Hype-y.

从哪里可以验证呢？在 docker hub 的网站上面，可以看一下关于 Windows 的镜像，在这里都没有收录 Windows 系列的操作系统镜像，都是一些应用程序。

[Docker Hub](https://link.zhihu.com/?target=https%3A//hub.docker.com/search%3Fq%3DWindows%26type%3Dimage%26page%3D1)

也就是说它比较适合 Linux 服务器的虚拟化，应用虚拟化。

在桌面虚拟化的领域，其实只有几家厂商可以做到技术完全自主化，这几家厂商分别是

```
Citrix 思杰 占有率46%   技术：XenServer
Vmware      占有率18%   技术：Vmware EXSI
微软        占有率未知  技术：hyper-v
Redhat      占有率未知  技术：LVM
华为        占有率未知  技术：openstack

```

在国内可能更多的听到是阿里云、腾讯云、或者华为云

我觉得桌面虚拟化更像 IASS 这种状态，虚拟化比较浅，可能只到操作系统这一层，下面是虚拟化几个方向的示意图，这种技术之后可能还得要装一些应用软件，比如 Windows 的 IIS，linux 的 Nginx，或者是 oracle 的数据库。

![屏幕截图-2023-02-02-163252](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230202/屏幕截图-2023-02-02-163252.7hssufoyglw0.jpg)


> docker 的目的主要还是用于应用容器, 而不是虚拟系统. 虽然 docker 提供了很多基础系统镜像, 但不是用来搭虚拟机用的, 只是提供一个应用的运行环境而已, 没法当作一个完整的系统. 就比如 docker 里不能运行 docker, 这是特性, 不是bug, 是设计目标与实现方案共同导致的结果.

docker核心之一就是在一个系统内核下, 运行多个不同的系统环境, docker容器启动后, 只启动了应用的进程, 不会启动其他系统进程。与虚拟机相比, 这样节约了重复启动多个系统内核的资源开销。

docker 跟虚拟机的使用场景不同, 要谨慎区分用途。

> doker只是自身秒级启动，并不能秒级启动各种镜像。Openstack不提供底层的资源虚拟化 华为的那个叫CNA

查资料的时候发现很多厂商都喜欢扯一些概念，什么 VDI、IDV、VOI、NGD，但是底层的技术实现干货很少。