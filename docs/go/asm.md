---
title: asm
layout: default
parent: go
has_children: false
---

# 符号

## 4个伪寄存器

- FP: frame pointer, 参数和本地变量
- PC: program counter: 跳转，分支
- SB: static base pointer: 全局符号
- SP: stack pointer: 栈顶

## 限制符号<>

表示只能在当前源文件中使用