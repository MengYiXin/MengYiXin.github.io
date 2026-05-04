---
title: Linux  CentOs 网络配置
date: 2023-01-31 20:05:21
tags:
	
	- Linux
---


最近对内部VMware虚拟机环境进行了配置，并且开启了一台win server2016作为跳板机，用Net路由转发实现上网。



对于每一台CentOS 的VM网络配置如下。最终成功ping通百度，但是需要上跳板机进行SSH连接。


 

 <div align="center"> <img src="https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/phone.bwxbq0uutjc.jpg" width = 400 /> </div>
 
 


![1675174566246](https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230131/1675174566246.1s34k1kirmf4.jpg)


 <div align="center"> <img src="https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/4.3b7o2i20owk0.jpg" width = 500 height = 300 /> </div>





最终实现上网：



 <div align="center"> <img src="https://cdn.staticaly.com/gh/MengYiXin/photo@master/20230218/zuihou.3ck42auqdq60.jpg" width = 400 /> </div>
 
>  注意，前提需要进入到   network-scripts目录下。配置完后重启网络服务。