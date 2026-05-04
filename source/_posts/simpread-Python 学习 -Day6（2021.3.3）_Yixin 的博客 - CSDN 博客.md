---
title: Python Day6
date: 2021-05-02 15:17:21
tags:
	
	- Python 
---


函数

```
# 函数
# 函数是可重复调用的代码段，能提高代码的复用率。
 
# 无参数
def print_hello():
    print ("hello")
 
 
print_hello()
 
 
# 带参数
def print_str(s):
    print(s)
    return s * 2
 
 
print_str("fuck")
 
 
# 带默认参数
def print_default(s="hello"):
    print(s)
 
 
print_default()
print_default("default")
 
 
# 不定长参数
def print_args(s, *arg):
    print(s)
    for a in arg:
        print(a)
    return
 
 
print_args("hello")
print_args("hello", "world", "1")
 
 
# 参数次序可以变
def print_two(a, b):
    print(a, b)
 
 
print_two(a="a", b="b")
print_two(b="b", a="a")
```