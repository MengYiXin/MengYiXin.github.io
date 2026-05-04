---
title: Linux CentOS创建网桥，实现KVM宿主机与虚拟机通讯
date: 2023-02-18 20:05:21
tags:
	
	- Linux
---



# Linux创建网桥，实现KVM宿主机与虚拟机通讯

由于业务需求，最近开始尝试在Linux(CentOS) 里面装虚拟机，而这个CentOs本身就是在vSphere中的虚拟机。也就是我们常说的，虚机化嵌套，套娃。业务主要涉及以下：


* 先在VMware vSphere环境里面起一台CentOS, 然后给其配置网络和SSH。
* 然后再给CentOs用命令行安装图形化桌面（使用KVM需要）。
* 之后再用网桥的方式，提前后面KVM要启动的虚机（jupiter—Ubuntu魔改）做网络层面的准备，KVM部署网络选项时候需要选自己新创建的网桥。
* 注意，可以用Xftp工具为虚拟机（CentOs）内部上传它所需要的镜像，如图。


![1](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/1.4xvpz1ozk880.jpg)


今天主要记录的是，如何 在centos7下创建网桥，解决KVM宿主机与虚拟机通讯？


## 第一种办法

1)	最开始我是通过B站视频，进行短暂学习，并且按照步骤搭建了一下环境。比如
1.  [centos7下kvm虚拟机的网络配置（桥接网卡模式）](https://www.bilibili.com/video/BV1M5411s7rP/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=069855d938307b0a82fdde6b3d8ddffa)

```bash
cd  /etc/sysconfig/network-scripts/   #（第一步先到网络目录下）
cp  ifcfg-ens192  ifcfg-wangq     #（先复制物料网卡信息-懒得写。再修改，wangq是随便起的名字）
ls                  			# (发现新增 ifcfg-wangq)
vim ifcfg-wangq    			 #（然后我们编辑一下网桥）
  TYPE=Bridge
  NAME不要，UUID不要，注意DEVICE后面一定要跟网桥的名字一样。NAME不要，UUID不要，注意DEVICE后面一定要跟网桥的名字一样。然后保存，退出。

vim ifcfg-ens192  # (然后物理网卡要加一句话。)
	BRIDGE="wangq"

/etc/init.d/network restart       #（重启网卡）

ip a                   			   #（可以看到网桥配置文件）

```
但是后来发现这样行不通，centos上的网络总是有问题，所以就准备找了其他办法。

## 第二种办法
2)	另一种办法是通过代码来实现网桥功能，我先修改了下面目录的配置文件（包括物理网卡配置nes192 和 新建的br0）：

```bash
/etc/sysconfig/network-scripts/
```

![1](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/1.1vyy1c8a9bmo.jpg)

![2](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/2.1c7u6m8zegcg.jpg)


![3](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/3.2qegeyma6qg0.jpg)


在此同时，我将之前的ens192的配置文件，也就是我物理网卡配置内容做个备份，以免遗忘。
```bash
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no

BOOTPROTO=static
PROXY_METHOD=none
BROWSER_ONLY=no

BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens192
UUID=2c658628-92f9-4c27-9cd6-6fbd2cfd585b

DEVICE=ens192
ONBOOT=yes
IPADDR=172.10.3.1
NETMASK=255.255.0.0
GATEWAY=172.10.0.1
DNS1=114.114.114.114

```

然后再通过下面的博客链接，的第五部分开始，进行相关配置。

2.  [Linux中KVM桥接的配置](https://www.cnblogs.com/heyongboke/p/10337447.html)

最终再参考这篇文章，最终实现配置网桥的目标。

3.  [Red Hat 7 创建网桥，解决KVM宿主机与虚拟机无法通讯](https://blog.51cto.com/cubix/1736750)


![2](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/2.kr5t51pm0mo.jpg)


配置完网卡后，重启虚机方能出现效果。






