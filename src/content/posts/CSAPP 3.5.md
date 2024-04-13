---
title: CSAPP 3.5
published: 2020-10-15
description: 数学运算和逻辑运算相关内容
image: ''
tags: [CSAPP]
category: Notes
draft: false
---



## CSAPP 3.5 数学运算与逻辑运算

在x86-64架构中，数学运算、逻辑运算都有专属的指令，并且大部分指令都能够适配四种不同的数据大小（后缀b、w、l、q）。

操作符能够被分为四种类型：

- 双目操作符（Binary）：需要两个操作数的操作符
- 单目操作符（Unary）：需要一个操作数的操作符
- 有效地址加载（Load Effective Address）：**该指令只有后缀q，不适配其余数据大小**
- 移位操作（Shift）



### 有效地址加载

有效地址加载（load effective address）指令格式为：
$$
leaq~~Src,~Dst
$$
这条指令实现的功能是从内存读取数据到寄存器，但并没有引用内存，而是将有效地址存入目标中。

除此以外，它还能够用来**计算一些简单的形如$x+k*y$的表达式**，其中的k需要是1、2、4、8中的一个。

例如计算$x*12$，则有汇编代码如下：

```
leaq (%rdi,%rdi,2), %rax	# t = x + 2*x
salq $2, %rax				# t << 2
```



### 双目操作符与单目操作符

双目操作符即是需要两个操作数的操作，其中的第二个操作数既是源，又要参与运算。

| 指令             | 效果       | 含义         |
| ---------------- | ---------- | ------------ |
| add       S, D   | D = D + S  | 加法运算     |
| sub       S, D   | D = D - S  | 减法运算     |
| imul      S, D   | D = D * S  | 整数乘法运算 |
| xor        S, D  | D = D ^ S  | 异或运算     |
| or          S, D | D = D \| S | 或运算       |
| and       S, D   | D = D & S  | 与运算       |

上表中的加减乘除与逻辑运算都属于二元运算指令。

第一个操作数可以是常数、寄存器或是内存，第二个操作数可以是寄存器或内存。规则与MOV的规则一致，两个操作数不能同时为内存。

单目操作符更加简单，只需要一个操作数作为源和目标。这里的操作数可以是寄存器或是内存，类似于C语言中的自增与自减语法。

| 指令        | 效果      | 含义     |
| ----------- | --------- | -------- |
| inc       D | D = D + 1 | 自增1    |
| dec      D  | D = D - 1 | 自减1    |
| neg      D  | D = -D    | 取负数   |
| not      D  | D = ~D    | 按位取反 |



### 移位运算

移位运算也需要两个操作数，其中第一个给定要移动的位数，第二个是待移动的数据。

| 指令         | 效果          | 含义        |
| ------------ | ------------- | ----------- |
| sal     k, D | D = D << k    | 左移k位     |
| shl     k, D | D = D << k    | 左移k位     |
| sar    k, D  | D = D >>$_A$k | 算术右移k位 |
| shr    k, D  | D = D >>$_L$k | 逻辑右移k位 |

第一个操作数k可以是常数，也可以是存在寄存器%cl中的数据。移位运算的相关指令只允许这个特定的寄存器作为操作数。

在x86-64架构中，对一个有w位的数据进行移位操作时，是根据寄存器%cl中的最低m位决定移动位数的，其中$2^m=w$，高位会被忽略。譬如当该寄存器中存的数据为0xFF时，salb指令只会移动7位，salw指令移动15位，sall指令移动31位，而salq指令移动63位。

左移指令有两个，效果相同，而右移则有算术右移和逻辑右移两种，效果与第二章中相同。



**课程内容至此为止，有关书中3.5节提及的特殊的运算并未展开，故本章整理也先就此打住，日后提及再进行补充。**