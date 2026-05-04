---
title: Python Day2
date: 2021-05-02 15:13:21
tags:
	
	- Python 
---


```
# 标准数据类型
import math
 
"""
Number(数字)
String(字符串)
List(列表)
Tuple(元组)
Set(集合)
Dictionary(字典)
可变数据（3个）：List(列表)，Dictionary(字典)，Set(集合)
代码风格整理： Ctrl + Alt +l
"""
 
a, b, c, d = 20, 5.5, True, 4 + 3j
 
print(type(a)), print(type(b)), print(type(c)), print(type(d))
# # ------------------------------------------------------------------------#
# # 此外还可以用 isinstance 来判断：
# a = 111
# isinstance(a, int)
#
# # isinstance 和 type 的区别在于：
# '''
# type()不会认为子类是一种父类类型。
# isinstance()会认为子类是一种父类类型。
# '''
#
#
# class A:
#     pass
#
#
# class B(A):
#     pass
#
#
# isinstance(A(), A)
#
# type(A()) == A
#
# isinstance(B(), A)
#
# type(B()) == A
# ------------------------------------------------------------------------#
 
# 数值运算：
 
5 + 4  # 加法
 
4.3 - 2  # 减法
```

![](https://img-blog.csdnimg.cn/20210225233251521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210225233313685.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70)