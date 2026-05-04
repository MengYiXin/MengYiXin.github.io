---
title: 从0开始装VSAN 
date: 2021-05-02 17:07:21
tags:
	
	- VMware
---




1. 请务必格式化好全部的硬盘，否则可能会出现配置好了但是不能使用的问题。  
2. 安装 VCSA 可能会卡 80% 参考网上解决。（有可能实际上是等待不够久，并不会出现进度条不会动的情况）  
3. 注意好安装环境的物理网络情况，确保过程中的通信。  
4.EVC 开启的话选择最新的 CPU 微架构

Partition1 - install ESXi  
进入 xcc 选择 ESXi 镜像，挂载，重启。根据 installer 提示完成 ESXi 安装（在本次实验环境下

Partition2 - deploy VCSA onto new vSAN cluster  
打开安装镜像，选择对应的安装程序，在选择 datastore 的时候选择安装在一个新的 vSAN 群集里，剩下的按照需求配置好

Parition3 - 修理剩下的错误

*   将其余的 vSAN 主机添加到群集中。
*   在每个主机上配置专用的 vSAN vmkernel。
*   将磁盘从其他主机添加到 vSAN 磁盘组。
*   一切正常运行后，确保运行了 vSAN Health Check，并且不要忽略任何错误！

[UPGRADING TO VCENTER 7.0 VIA CLI](https://thewificable.com/2018/04/26/deploy-vcsa-6-7-onto-new-vsan-6-7-cluster/)  
[将 vSAN 群集从一个 vCenter Server 移至另一个 vCenter Server (2151610)](https://kb.vmware.com/s/article/2151610?lang=zh_cn)

[如果 vCenter Server 虚拟机属于同一群集，如何启用 EVC（1013111）](https://kb.vmware.com/s/article/1013111)