---
title: Python Day7
date: 2021-05-02 15:18:21
tags:
	
	- Python 
---




# 集合

```
# 模块
# 通常来说，比较正规的工程不会把所有代码放在一个py文件里，我们会把代码拆成各个模块，分别调用。对python来说，拆成各个模块可以看做拆成各个py文件。
 
 
# 搜索路径
# 通常来说，同文件夹下的py文件可以直接import。
 
def print_hello():
    print("hello")
 
 
# 我们把这个保存至hello.py
import hello
 
hello.print_hello()
 
# 在run.py里import，然后调用print_hello() 目录结构
# ...../
 
hello.py
run.py
 
# hello.py和run.py在同一目录下，可以直接import 如果在不同路径下，可以在sys.path里手动加入你想import的路径
 
import sys
 
sys.path.append('/home/yixin/course')
import hello
 
hello.print_hello()
# 如果run.py不在/home/yixin/course文件夹下，手动加入这个路径，就可以import这个路径下的hello.py
 
# ----------------------------------------------------------------------------------------------------
# 通常一个工程不可能只有一层目录结构，并且也不会一个一个path去append到sys里，常用的做法是包，一个目录及其子目录组成的一个包（可以看做一个库）。 举个例子
#
"""/home/yixin/course
├── __init__.py
├── __init__.pyc
├── m1
│   ├── b.py
│   ├── b.pyc
│   ├── __init__.py
│   ├── __init__.pyc
│   └── m1_1
│       ├── a.py
│       ├── a.pyc
│       ├── __init__.py
│       └── __init__.pyc
└── m2
    ├── __init__.py
    └── run.py
"""
```