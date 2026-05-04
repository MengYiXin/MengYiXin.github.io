---
title: Python Day3
date: 2021-05-02 15:14:21
tags:
	
	- Python 
---


```
# Python的数据结构有三种:列表、元组和字典
 
 
# 列表（list）
# 初始化：[],[1,3,7],['a','c'],[1,'s','des',256]等
# 1，增加：append(value),extend(list2),insert(i,value)
 
mylist = [1, 3, 7]
mylist.append('a')
mylist.insert(2, 'fff')
print(mylist)
 
newList = ['abc', 'kkk', 123]
mylist.extend(newList)
print(mylist)
 
# 2,删除：pop([i]）,remove（value）
# i 可以是负数
# i 超出范围会报out of range错误
# remove只会移除第一个遇到的值
# pop 有返回值，remove没有
 
mylist = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
a = mylist.pop()
print(a)
print(mylist)
b = mylist.pop(2)
print(b)
print(mylist)
c = mylist.pop(-1)
print(c)
print(mylist)
newList = ['a', 'b', 'b', 'c', 'd']
k = newList.remove('b')
print(k)
print(newList)
 
# 3.查询:index(value, [start, [stop]])　　
# ---返回列表中第一个出现的值为value的索引，如果没有，则异常 ValueError
 
mylist = [1, 2, 5]
a = mylist.index(2)
print(a)
'b = mylist.index(3)  # 此处会报错，返回以下信息'
 
# Traceback (most recent call last):
#   File "C:/Users/mengyx3/Desktop/Python learning/learning-python/mengyixin/day3.py", line 44, in <module>
#     b = mylist.index(3)
# ValueError: 3 is not in list
 
"""4.修改:list没有直接修改对应元素的方法，
   只能先找到目标元素所在位置，然后直接赋值"""
 
mylist = ['a', 'k', 'm']
myIndex = mylist.index('k')
mylist[myIndex] = 'fff'
print(mylist)
 
# 5.排序:sort()
 
mylist = [6, 23, 7, 39, 1, 90]
mylist.sort()
print(mylist)
 
# 6.反转：reverse()
 
mylist = ['f', 'm', 'a', 'z']
mylist.reverse()
print(mylist)
 
# 清空：clear()
 
mylist = [1, 2, 4]
mylist.clear()
print(mylist)
 
```

![](https://img-blog.csdnimg.cn/20210226181007597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70)