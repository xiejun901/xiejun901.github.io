title: C++中的返回值优化
date: 2015-10-20 21:11:01
categories: C++
tags: [C++, RVO, 返回值优化, 函数调用, 栈帧]
---

之前面试的时候被问到过C++返回值优化的问题，然后发现自己没有理得很清楚，因此仔细去看一下理解一下并且写一篇博客记录一下
记录的大部分内容应该会来源于[RVO V.S. std::move--Zhao Wu](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=en)以及[知乎网友的一些讨论](http://www.zhihu.com/question/29511959)

## C/C++栈帧介绍
先介绍一下x86-64中的16个64位寄存器

![](http://7q5fny.com1.z0.glb.clouddn.com/blogx86-64寄存器2.png)
图片来自[X86-64寄存器和栈帧](http://www.searchtb.com/2013/03/x86-64_register_and_function_frame.html)

	%rax: 作为函数返回值使用
	%rdi，%rsi，%rdx，%rcx，%r8，%r9: 用作函数参数，一次对应第一参数，第二参数
	%rsp: 堆栈指针，指向帧顶
	
	%rbx，%rbp，%r12，%r13，%14，%15 用作数据存储，遵循被调用者使用规则，
	简单说就是随便用，调用子函数之前要备份它，以防他被修改，rbp一般可以用来保存堆栈底指针
	%r10，%r11 用作数据存储，遵循调用者使用规则，简单说就是使用之前要先保存原值

对于以下一段程序，用g++ 编译成汇编
	
```C++
int fun(int i)
{
	return i*2;
}
int main()
{
	int i=10;
    int j=fun(i);
    return 0;
}

```

```s

	.file	"test_stackfram.cpp"
	.text
	.globl	_Z3funi
	.type	_Z3funi, @function
_Z3funi:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	%edi, -4(%rbp)
	movl	-4(%rbp), %eax
	addl	%eax, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	_Z3funi, .-_Z3funi
	.globl	main
	.type	main, @function
main:
.LFB1:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movl	$10, -4(%rbp)
	movl	-4(%rbp), %eax
	movl	%eax, %edi
	call	_Z3funi
	movl	%eax, -8(%rbp)
	movl	$0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	main, .-main
	.ident	"GCC: (GNU) 5.2.0"
	.section	.note.GNU-stack,"",@progbits

```

从main函数看，首先进入main函数将%rbp入栈，然后将%rsp的值赋给%rbp，开始一个新的栈帧，此时帧顶栈底在同一个位置，表示了main函数的栈
接下来会把局部i=10存在main函数的栈帧中，然后是通过寄存器%edi来传递参数调用fun函数，函数返回值会存在eax中，然后再将函数返回值置于main函数栈帧中的j局部变量，i位于(%rbp-4)的位置,位于j(%rbp-8)的位置

看下fun函数，进入fun函数之后，会首先保存%rbp，即保存好调用方的栈帧的栈底指针，然后会将%rbp赋值为%rsp，开始一个新的栈帧，这个栈帧会用来存放fun函数的局部变量等等，如果再有一层调用，在被调用方会继续保存fun函数的堆栈底指针，并开辟新的栈帧



