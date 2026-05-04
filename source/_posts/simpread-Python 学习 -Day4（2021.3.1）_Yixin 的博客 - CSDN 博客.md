---
title: Python Day4
date: 2021-05-02 15:15:21
tags:
	
	- Python 
---


```
# 字典（dict）
# 　定义:键值对集合
#
# 　　初始化:{}, {'1' : 'abc', '2' : 'def'}
#
# 　　1.增加:单个数据直接赋值
 
#    update(dict2)　　---把dict2的元素加入到dict中去，键字重复时会覆盖dict中的键值
 
myd = {}
 
myd['1'] = 'one'
 
print(myd)
 
myd1 = {1: 'one', 2: 'two'}
myd2 = {1: 'one', 3: 'three'}
myd1.update(myd2)
print(myd1)
 
#     2,删除:pop(key, [default])　　---若字典中key键存在，删除并返回dict[key]，若不存在，且未给出default值，引发KeyError异常
#
# 　　　　　popitem()　　---删除任意键值对，并返回该键值对，如果字典为空，则产生异常KeyError
#
# 　　　　　clear()  ---略
 
myd = {1: "one", 2: "two", 3: "there"}
item = myd.pop(1)
print(item)
print(myd)
newItem = myd.popitem()
print(newItem)
 
print(myd)
 
#     3,查询：get(key,[default])　　---返回字典dict中键key对应值，如果字典中不存在此键，则返回default 的值(default默认值为None)
#
# 　　　　　　items()　　---返回一个包含字典中(键, 值)对元组的列表
#
# 　　　　　　keys()　　---返回一个包含字典中所有键的列表
#
# 　　　　　　values()　　---返回一个包含字典中所有值的列表
#
myd = {1: "one", 2: "two", 3: "there"}
print(myd.get(2))
 
print(myd.items())
 
print(myd.keys())
print(myd.values())
```

![](https://img-blog.csdnimg.cn/20210301174315682.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70)