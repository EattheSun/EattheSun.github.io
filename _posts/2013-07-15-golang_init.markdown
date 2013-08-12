---
layout: post
title: Golang 在 main() 执行前干了什么?
tags:
   - golang
---

和其他语言一样，在主程序执行前总有一堆麻烦的破事被隐藏了。比如C语言的main被执行之前，又比如python。

在包的导入上，golang 比 C 语言更有优势的一个地方在于，golang 不会一个重复被包含的包导入进来。

导入一个依赖包的过程为:

- step1. 导入这个依赖包的依赖包
- step2. 生成依赖包级别的常量
- step3. 生成依赖包级别的变量
- step4. 调用依赖包中的init()函数

一个程序的启动过程(即main包)：

- step1. 导入依赖包(依赖包的依赖包递归导入)
- step2. 生成本包级别的常量
- step3. 生成本包级别的变量
- step4. 调用init()函数(因此init()函数中可以引用常量和本包级别的变量)
- step5. 调用main()函数(因此init()函数中不可以引用main()函数中的变量)

目前，golang 的一个包只允许包含一个init()函数.

例如:

依赖包为:

	package mypac

	func init() {
		println("Hello, I'm mypac init.")
	}

主程序为:

	package main

	import mypac

	func init() {
		println("Hello, I'm main init.")
	}

	func main() {
		println("Hello, I'm main.")
	}

Run it:
	
	Hello, I'm mypac init.
	Hello, I'm main init.
	Hello, I'm main.
	

