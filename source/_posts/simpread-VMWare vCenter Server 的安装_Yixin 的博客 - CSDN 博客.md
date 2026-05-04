---
title: VMWare vCenter Server 的安装
date: 2021-05-02 17:03:21
tags:
	
	- VMware
---



![](https://img-blog.csdnimg.cn/2020081314570043.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145705267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/2020081314572026.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145724929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145731940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145740243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145748950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/2020081314575770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145804971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145810474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145816472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145821286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145828354.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145836692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145844835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145849977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145855385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145900945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145906757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145912890.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813145919634.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
从 ESXI 中，删除 Vcenter 虚拟机本身，然后全部进行重装一遍。熟悉流程  
![](https://img-blog.csdnimg.cn/20200813145932317.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
2020/7/22

![](https://img-blog.csdnimg.cn/20200813145952729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
具体虚拟机进入操作系统的密码

![](https://img-blog.csdnimg.cn/20200813150003859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813150009507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813150014997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813150020403.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)  
![](https://img-blog.csdnimg.cn/20200813150025814.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70#pic_center)