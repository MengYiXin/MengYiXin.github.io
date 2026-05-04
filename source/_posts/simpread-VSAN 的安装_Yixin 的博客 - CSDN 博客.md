---
title: VMWare VSAN 的安装
date: 2021-05-02 17:05:21
tags:
	
	- VMware
---



> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_36650546/article/details/107980254?spm=1001.2014.3001.5501)

![](https://img-blog.csdnimg.cn/20200813150837585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813150846183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151038205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
将 VCenter 安装在任何一台主机上（192.168.30.2，192.168.30.3，192.168.30.4）  
![](https://img-blog.csdnimg.cn/2020081315110279.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151108319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
这周（7 月 28 号）教我了 VSAN 的知识，交我一个任务作业。他给我了 3 个 XCC 地址，有 3 台装了 ESXI 的主机，让我装 VSAN。跟以往不同的地方是，这次让我把 Vcenter 装在这三台主机中的其一，主机里边。以往都是装在主机外的，用来管理主机。就遇到这个问题。

![](https://img-blog.csdnimg.cn/20200813151153128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151158991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
2020 年 0802

刚开始找不到 VSAN，6.5 有 html5 和 flash 两种访问方式，html5 在 6.5 中功能不全。

![](https://img-blog.csdnimg.cn/2020081315122168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151225856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151231541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813151241858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)