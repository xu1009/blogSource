---
title: golang学习
date: 2017-06-23 15:37:14
tags: golang
---
### go语言
先来一段代码感受一下
```go
package main

import "fmt"

func main()  {
	fmt.Print("hello world")
}
```
> 分析一下这段代码
*  包是main这个也是有意义的，包是main的表示该段代码可以独立执行
*  main函数和正常的程序一样，程序运行时首先执行的，如果有init函数就执行init
*  标识符首字母的大小写决定了该变量的可见性，大写就是public，小写是private
***
> go语言也需要提前声明变量

```go
var a int = 1
var a = 1
a := 10
```
> 常量const，不能被修改的量