---
title: 使用Linux的iptables实现路由转发
date: 2023-02-03 20:05:21
tags:
	
	- Linux
---



# 使用linux的iptables实现路由的转发

## 开启ip包转发功能
linux默认关闭该功能
```bash
vim /etc/sysctl.conf

#加入或修改
net.ipv4.ip_forward = 1
```
重启
```bash
reboot
```

## 开启NAT功能
```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```
将非本机地址的ip包修改源地址后从连网的网口发出去，enp0s3修改为自己的外网网卡
重启会发现配置失效。下面有一种方案可以让配置永久生效，但是不是很优雅，但是可以避免使用iptables-services
```bash
iptables-save > /etc/sysconfig/iptables
```
然后修改/etc/rc.d/rc.local，可以在启动时自动执行该命令
```vim
#将下面一行加入文件最后
iptables-restore < /etc/sysconfig/iptables
```
将该文件设为可执行文件
```bash
chmod +x /etc/sysconfig/iptables
```
这样重启时就可以保持该设置了。


## NAT本机网卡设置
```bash
# 将ifcfg-enp0s8网卡配置的下列内容修改为你的信息
IPADDR=172.10.1.1
MASK=255.255.0.0
```

## 其它机器网卡设置
```bash
# 将网关gateway设置为上边nat主机的地址
IPADDR=172.10.1.2
MASK=255.255.0.0
GATEWAY=172.10.1.1
DNS1=114.114.114.114
```