---
title: go语言学习
date: 2021-05-02 17:01:21
tags:
	
	- golang
---



第二天
---

这里加了命令行参数，他是以输出的形式输出了命令行参数。

[C:\Users\mengyx3\AppData\Local\Temp\go-build833694214\b001\exe\hello_world.exe 是二进制的命令， chao 是命令行参数了。

![](https://img-blog.csdnimg.cn/2020110900003021.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NjUwNTQ2,size_16,color_FFFFFF,t_70)

package main  
import (  
   "fmt"  
   "os"  
)  
func main() {  
   fmt.Println(os.Args)  
   fmt.Println("Hello World")  
   os.Exit(-1)  
}

运行程序，我们看到命令行参数被放在了 hello world 的后面。

编写测试程序

1.  源码文件以_test 结尾：xxx_test . go
2.  测试方法名以 Test 开头：func TestXXX(t *testing.T) {…}

斐波那契数列

也可以改写成

还可以改写成

刚刚都是用的 fmt.Print 来输出的，其实在单元测试可以用 t.Log 来输出。

Go 语言里交换变量的值写法简洁，可以在一句赋值语句里边 对多个变量进行赋值。

位运算

位运算结果这里，应该是三个 true , 也就是 true true true 。而不是 true false true ?

当前播放：05 | 变量，常量以及与其他语言的差异

刚刚上面是 a:=7 0111     我们换成 a:=1 0001

第三天
---

1.  数据类型和指针，不支持隐性的数据类型转换。需要显性数据类型转换。

1.  Go 语言可以支持指针类型，但是不支持指针运算。

1.  Go 的字符串是值类型，默认初始化零值是空字符串，而不是空。

第四天
---

用 == 比较数组

1.  相同维数且含有相同个数元素的数组才可以比较
2.  每个元素都相同才相等。

1.  按位置零

第五天
---

1.  条件和循环