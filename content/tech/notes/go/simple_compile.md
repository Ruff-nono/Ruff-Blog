---
title: "一个简单的go代码对应的汇编学习"
slug: ""
date: 2023-08-25T11:25:54+08:00
lastmod: 2023-08-25T11:25:54+08:00
author: ["路非非"]
tags: # 标签
-
series:
-
description: ""
weight:
draft: true # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

## 编译
一个简单的代码
```go
package main

func a() int { return 1 }

func main() {
	print(a())
}
```
通过`go tool compile`编译go代码（flag: compile link之类后面再学）
```shell
go tool compile -S main,go
```
可以看到两个函数的汇编，分别表示`main.a`和`main.main`

```
main.a STEXT nosplit size=6 args=0x0 locals=0x0 funcid=0x0 align=0x0
	0x0000 00000 (main,go:3)	TEXT	main.a(SB), NOSPLIT|ABIInternal, $0-0
	0x0000 00000 (main,go:3)	FUNCDATA	$0, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
	0x0000 00000 (main,go:3)	FUNCDATA	$1, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
	0x0000 00000 (main,go:3)	MOVL	$1, AX
	0x0005 00005 (main,go:3)	RET
	0x0000 b8 01 00 00 00 c3                                ......
```
```
main.main STEXT size=59 args=0x0 locals=0x10 funcid=0x0 align=0x0
	0x0000 00000 (main,go:5)	TEXT	main.main(SB), ABIInternal, $16-0
	0x0000 00000 (main,go:5)	CMPQ	SP, 16(R14)
	0x0004 00004 (main,go:5)	PCDATA	$0, $-2
	0x0004 00004 (main,go:5)	JLS	52
	0x0006 00006 (main,go:5)	PCDATA	$0, $-1
	0x0006 00006 (main,go:5)	SUBQ	$16, SP
	0x000a 00010 (main,go:5)	MOVQ	BP, 8(SP)
	0x000f 00015 (main,go:5)	LEAQ	8(SP), BP
	0x0014 00020 (main,go:5)	FUNCDATA	$0, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
	0x0014 00020 (main,go:5)	FUNCDATA	$1, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
	0x0014 00020 (main,go:6)	PCDATA	$1, $0
	0x0014 00020 (main,go:6)	CALL	runtime.printlock(SB)
	0x0019 00025 (main,go:6)	MOVL	$1, AX
	0x001e 00030 (main,go:6)	NOP
	0x0020 00032 (main,go:6)	CALL	runtime.printint(SB)
	0x0025 00037 (main,go:6)	CALL	runtime.printunlock(SB)
	0x002a 00042 (main,go:7)	MOVQ	8(SP), BP
	0x002f 00047 (main,go:7)	ADDQ	$16, SP
	0x0033 00051 (main,go:7)	RET
	0x0034 00052 (main,go:7)	NOP
	0x0034 00052 (main,go:5)	PCDATA	$1, $-1
	0x0034 00052 (main,go:5)	PCDATA	$0, $-2
	0x0034 00052 (main,go:5)	CALL	runtime.morestack_noctxt(SB)
	0x0039 00057 (main,go:5)	PCDATA	$0, $-1
	0x0039 00057 (main,go:5)	JMP	0
	0x0000 49 3b 66 10 76 2e 48 83 ec 10 48 89 6c 24 08 48  I;f.v.H...H.l$.H
	0x0010 8d 6c 24 08 e8 00 00 00 00 b8 01 00 00 00 66 90  .l$...........f.
	0x0020 e8 00 00 00 00 e8 00 00 00 00 48 8b 6c 24 08 48  ..........H.l$.H
	0x0030 83 c4 10 c3 e8 00 00 00 00 eb c5                 ...........
	rel 21+4 t=7 runtime.printlock+0
	rel 33+4 t=7 runtime.printint+0
	rel 38+4 t=7 runtime.printunlock+0
	rel 53+4 t=7 runtime.morestack_noctxt+0
go.cuinfo.producer.<unlinkable> SDWARFCUINFO dupok size=0
	0x0000 72 65 67 61 62 69                                regabi
go.cuinfo.packagename.main SDWARFCUINFO dupok size=0
	0x0000 6d 61 69 6e                                      main
go.info.main.a$abstract SDWARFABSFCN dupok size=11
	0x0000 05 6d 61 69 6e 2e 61 00 01 01 00                 .main.a....
main..inittask SNOPTRDATA size=24
	0x0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
	0x0010 00 00 00 00 00 00 00 00                          ........
gclocals·g2BeySu+wFnoycgXfElmcg== SRODATA dupok size=8
	0x0000 01 00 00 00 00 00 00 00                          ........
```


## main.a
这是一个非常简单的函数，将常数1加载到寄存器AX中，然后返回。

```
TEXT	main.a(SB), NOSPLIT|ABIInternal, $0-0
```
定义了一个名为 main.a 的函数，SB 是基址寄存器， NOSPLIT 表示不进行栈分割优化，ABIInternal 表示该函数只在内部使用，$0-0 表示没有参数和局部变量。

>NOSPLIT 是一个Go汇编中的标志，用于指示编译器不要对包含这个标志的函数进行栈分割优化。栈分割是一种优化技术，目的是将函数调用所需的栈空间从堆栈分离出来，以减少栈帧大小，从而在某些情况下提高执行效率。
> 
>然而，在一些情况下，栈分割可能会导致一些问题。例如，栈分割后可能需要进行额外的栈管理工作，这会在一些小函数中引入不必要的开销。在这些情况下，你可能希望禁用栈分割优化，以便生成更简单、更高效的代码。这就是 NOSPLIT 标志的作用。
>
>当你在一个函数的定义中使用了 NOSPLIT 标志，编译器会避免对该函数进行栈分割，从而确保函数的栈帧不会被分离，减少不必要的开销。

```
FUNCDATA $0, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
```
FUNCDATA是汇编中的一个伪指令,用于存储与函数相关的元数据信息。  
整句表示将函数的局部变量初始化为零值。这个初始化信息是在编译时由编译器生成的，当编译器需要为函数的局部变量分配栈空间时，它会在栈上进行零值初始化。

```
FUNCDATA	$1, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
```
表示将函数的局部变量分配栈空间并初始化为非零值。

```
MOVL	$1, AX
```
将常数值1加载到AX寄存器中。

```
RET
```
函数返回

```
0x0000 b8 01 00 00 00 c3                                ......
```
十六进制表示的字节序列,每个字节都表示一个机器指令或数据。  
b8 是MOVL的操作码，01 00 00 00表示值1的字节表示。c3 是一条RET指令，表示函数的返回。  
将常数值1加载到寄存器中，然后返回。

## main.main
```
  TEXT	main.main(SB), ABIInternal, $16-0
```
定义了一个名为 `main.main` 的函数，`SB` 是基址寄存器，`ABIInternal` 表示该函数只在内部使用，`$16-0` 表示函数需要16字节的栈空间。

```
  CMPQ	SP, 16(R14)
  JLS	52
```
比较栈指针(SP)和16(R14)的值，如果比较结果小于，跳转到标签52处。  
`CMPQ`比较指令，`SP` 栈指针寄存器，指向当前栈的顶部，`16(R14)`表示从寄存器 R14（调用者保存的寄存器之一）所指向的地址往后偏移16个字节。通常用于访问函数参数和返回地址。   
`JLS` jump if less  
这两条指令的组合在函数开始时进行栈溢出检查，如果栈指针小于某个阈值（通常是检查栈空间是否足够以避免栈溢出），就会跳转到标签 52 处，这可能是函数中的一个错误处理分支。如果栈空间足够，程序会继续执行接下来的指令。这是一种常见的栈溢出保护机制。

```
	SUBQ $16, SP
	MOVQ BP, 8(SP)
	LEAQ 8(SP), BP
```
