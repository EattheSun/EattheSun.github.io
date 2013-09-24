---
layout: post
title: golang中的交叉编译
tags:
   - golang
   - cgo
   - 交叉编译
___


交叉编译是个很常见的需求，其实在 golang 中进行交叉编译是很方便的事情，但是还是有一些注意事项需要说明。

**注：本文撰写时最新版本为go 1.1.2**


#### 交叉编译所涉及到的东西

1. 环境变量GOOS, GOARCH, CGO_ENABLED, 以及可能需要使用到 GOARM
2. go 源码包（$GOROOT）中的 src/make.bash 脚本

其他就没啥东西了，简单吧。

#### 顺带说下怎么安装go

    step1. 获得源码（各种途径，略）
    step2. cd go/src
    step3. ./all.bash
    
完成调用`all.bash`之后，go的编译就ok了，但是注意这一步只是针对本机所属平台进行的编译，这一步完成后的go是不能用来编译其他平台的程序的。

#### 增加go对其他平台的支持

进入 $GOROOT/src 目录，执行:

    sudo CGO_ENABLED=0 GOOS=<GOOS> GOARCH=<GOARCH> ./make.bash
    
执行完之后会发现$GOROOT/pkg下多了对应的\<GOOS\>_\<GOARCH\>的目录，下面即针对这个平台的编译文件。

**PS**：如果不想这么麻烦地一个一个编译各个平台，那么可以看 [这里](https://github.com/davecheney/golang-crosscompile)，有人写了一个脚本可以一次性增加完对所有平台的编译支持，非常方便。同时还增加了各个平台对应的go命令，可以直接使用，编译出相应平台的程序。文档参考 [这里](http://dave.cheney.net/2012/09/08/an-introduction-to-cross-compilation-with-go)。


#### 如何进行交叉编译

在添加了对某个平台的支持后，设置好 GOOS 和 GOARCH 环境变量既可。例如：

    export GOOS=windows
    export GOARCH=amd64
    
OK，然后再执行`go build`之类的指令，就会直接编译出windows_amd64平台的程序了。


#### CGO怎么办？

虽然目前 golang 对交叉编译支持地很好，但是这是对纯go语言实现的工程而言的。如果一个项目包含了C代码——即需要CGO才能编译——那么交叉编译就会失败。

你会发现这样一个错误：
    
    cannot open file: /home/jenkins/devtools2/go/pkg/windows_386/runtime/cgo.a
    

这个错误是说，工程需要使用cgo模块来将C的代码融合到go代码中去，但是却在go源码中找不到相应平台对应的cgo静态库（即cgo.a）。

出现这个错误的原因在于，go的cgo是依赖于本地的C编译器的，C编译器会直接参与到将C代码编译到go代码这个过程中，但是目前C编译器是直接硬编码在gcc中，而gcc是只能默认根据本地平台选定。所以，go目前无法支持在A平台上编译B平台上的C代码。

所以，你会发现，`$GOROOT/pkg/$GOOS_$GOARCH/runtime/`目录下，只有本地平台所属的目录中有cgo.a这个文件。其他平台对应的目录下是没有的。

##### 解决办法

解决办法其实很简单，既然说找不到这个cgo.a库，那么我就从别的平台上编好了直接移过来不就行了么？Yeah，就是这么干的，完全可行。
