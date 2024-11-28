---
title: asm
layout: default
parent: go
has_children: false
---

# 符号

## 4个伪寄存器

- FP: frame pointer, 参数和本地变量，FP寄存器指向函数参数。0(FP)是第一个参数，8(FP)是第二个参数(64-bit machine). first_arg+0(FP)表示把第一个参数地址绑定到符号 first_arg, 这个与SB的含义不同。
- PC: program counter: 跳转，分支
- SB: static base pointer: 全局符号，SB寄存器表示全局内存起点，foo(SB) 表示 符号foo作为内存地址使用
- SP: stack pointer: 栈顶，SP寄存器表示栈指针，指向 top of local stack frame, 所以 offset 都是负数，范围在 [ -framesize, 0 ), 例如 x-8(SP). 对于硬件寄存器名称为SP的架构，x-8(SP) 表示虚拟栈指针寄存器， -8(SP) 表示硬件 SP 寄存器.

## 说明
FP: 使用形如 symbol+offset(FP) 的方式，引用函数的输入参数。
    例如 arg0+0(FP)，arg1+8(FP)，使用 FP 不加 symbol 时，无法通过编译，在汇编层面来讲，symbol 并没有什么用，加 symbol 主要是为了提升代码可读性。
    另外，官方文档虽然将伪寄存器 FP 称之为 frame pointer，实际上它根本不是 frame pointer，按照传统的 x86 的习惯来讲，frame pointer 是指向整个 stack frame 底部的 BP 寄存器。
    假如当前的 callee 函数是 add，在 add 的代码中引用 FP，该 FP 指向的位置不在 callee 的 stack frame 之内，而是在 caller 的 stack frame 上。具体可参见之后的 栈结构 一章。
PC: 实际上就是在体系结构的知识中常见的 pc 寄存器，在 x86 平台下对应 ip 寄存器，amd64 上则是 rip。
    除了个别跳转之外，手写 plan9 代码与 PC 寄存器打交道的情况较少。
SB: 全局静态基指针，一般用来声明函数或全局变量，在之后的函数知识和示例部分会看到具体用法。
SP: plan9 的这个 SP 寄存器指向当前栈帧的局部变量的开始位置，使用形如 symbol+offset(SP) 的方式，引用函数的局部变量。
    offset 的合法取值是 [-framesize, 0)，注意是个左闭右开的区间。
    假如局部变量都是 8 字节，那么第一个局部变量就可以用 localvar0-8(SP) 来表示。
    这也是一个词不表意的寄存器。
    与硬件寄存器 SP 是两个不同的东西，在栈帧 size 为 0 的情况下，伪寄存器 SP 和硬件寄存器 SP 指向同一位置。
    手写汇编代码时，如果是 symbol+offset(SP) 形式，则表示伪寄存器 SP。
    如果是 offset(SP) 则表示硬件寄存器 SP。
    务必注意，对于编译输出(go tool compile -S / go tool objdump)的代码来讲，目前所有的 SP 都是硬件寄存器 SP，无论是否带 symbol。

## 限制符号<>

表示只能在当前源文件中使用


# 指令 

## 长度

MOVB $1, DI      // 1 byte
MOVW $0x10, BX   // 2 bytes
MOVD $1, DX      // 4 bytes
MOVQ $-10, AX    // 8 bytes

## 常见指令

ADDQ  AX, BX   // BX += AX
SUBQ  AX, BX   // BX -= AX
IMULQ AX, BX   // BX *= AX

## 跳转

// 无条件跳转
JMP addr   // 跳转到地址，地址可为代码中的地址，不过实际上手写不会出现这种东西
JMP label  // 跳转到标签，可以跳转到同一函数内的标签位置
JMP 2(PC)  // 以当前指令为基础，向前/后跳转 x 行
JMP -2(PC) // 同上

// 有条件跳转
JZ target // 如果 zero flag 被 set 过，则跳转

##  X64 和 plan9 中的对应关系:

|  X64   | plan9 |
|   :----:  |  :----:  |
| rax  | AX |
| rbc  | BX |
| rcx  | CX |
| rcx  | DX |
| rdi  | DI |
| rsi  | SI |
| rbp  | BP |
| rsp  | SP |
| r8  | R8 |
| r9  | R9 |
| r10  | R10 |
| r11  | R11 |
| r12  | R12 |
| r13  | R13 |
| r14  | R14 |
| rip  | PC |


## 变量声明

symbol 为变量在汇编语言中对应的标识符，offset 是符号开始地址的偏移量，width 是要初始化内存的宽度大小，value 是要初始化的值。
`DATA    symbol+offset(SB)/width, value`
比如：
`DATA ·Id+0(SB)/1,$0x37`
`DATA ·Id+1(SB)/1,$0x25`

变量定义好之后需要导出以供其它代码引用。Go 汇编语言提供了 GLOBL 命令用于将符号导出
`GLOBL symbol(SB), width`
比如：
`GLOBL ·Id, $8`

```asm
DATA age+0x00(SB)/4, $18  // forever 18
GLOBL age(SB), RODATA, $4

DATA pi+0(SB)/8, $3.1415926
GLOBL pi(SB), RODATA, $8

DATA birthYear+0(SB)/4, $1988
GLOBL birthYear(SB), RODATA, $4


DATA bio<>+0(SB)/8, $"oh yes i"
DATA bio<>+8(SB)/8, $"am here "
GLOBL bio<>(SB), RODATA, $16
```

## 使用时.s文件和.go文件变量互通

- refer.go
```go
package main

var a = 999
func get() int

func main() {
    println(get())
}
```
- refer.s
  
```asm
#include "textflag.h"

TEXT ·get(SB), NOSPLIT, $0-8
    MOVQ ·a(SB), AX
    MOVQ AX, ret+0(FP)
    RET
```