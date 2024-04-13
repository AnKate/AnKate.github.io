---
title: CSAPP 3.4
published: 2020-10-14
description: 访问数据和寄存器相关内容
image: ''
tags: [CSAPP]
category: Notes
draft: false 
---



## CSAPP 3.4 访问信息

在x86-64架构下，CPU有一组16个能够存储64位的寄存器，它们能够用来管理栈、传递函数所用到的变量、从函数返回值或是存储局部或全局变量。



#### 操作数指示符

大多数指令中会有一个或多个操作数，用于指示指令的源头和目标位置。

操作数可以分为三类：

- Immediate，在中文版中译作**立即数**，实际即是常数的意思。通常用'$'符号后接一个标准C语言表示法的常数。
- Register，即寄存器，表示的是寄存器中的内容。用$r_a$表示该寄存器，$R[r_a]$表示该寄存器中的内容的值。
- Memory，即内存，需要通过计算得出的地址访问内存中的位置，该地址通常被称作**有效地址**。用$M_b[Addr]$表示一个被存储在地址$Addr$中的大小为b(单位:$Byte$)的值，下标可省略。、

下表展示了不同的表示形式对应的操作数：

| 操作数类型 | 表示形式         | 操作数实际数值           | 名称              |
| ---------- | ---------------- | ------------------------ | ----------------- |
| Immediate  | $Imm             | $Imm$                    | Immediate         |
| Register   | $r_a$            | $R[r_a]$                 | Register          |
| Memory     | $Imm$            | $M[Imm]$                 | Absolute          |
| Memory     | $(r_a)$          | $M[R[r_a]]$              | Indirect          |
| Memory     | $Imm(r_b)$       | $M[Imm+R[r_b]]$          | Base+displacement |
| Memory     | $(r_b,r_i)$      | $M[R[r_b]+R[r_i]]$       | Indexed           |
| Memory     | $Imm(r_b,r_i)$   | $M[Imm+R[r_b]+R[r_i]]$   | Indexed           |
| Memory     | $(,r_i,s)$       | $M[R[r_i]*s]$            | Scaled indexed    |
| Memory     | $Imm(,r_i,s)$    | $M[Imm+R[r_i]*s]$        | Scaled indexed    |
| Memory     | $(r_b,r_i,s)$    | $M[R[r_b]+R[r_i]*s]$     | Scaled indexed    |
| Memory     | $Imm(r_b,r_i,s)$ | $M[Imm+R[r_b]+R[r_i]*s]$ | Scaled indexed    |

其中$s$是比例因子，必须是1,2,4,8中的一个。

从表中可以看出，除去最基础的几种直接访问外，通常用到的较多的几种寻址方法中都会带有偏移量以及比例因子，如结构体中的元素、或是访问数组元素。



#### 数据转移指令

##### 相同数据类型的转移

将数据从一个位置转移到另一个位置时将会用到数据转移指令，其中后缀对应的数据类型与前几张中所涉及的并无出入，具体情况见下表：

| 指令    | 描述                    |
| ------- | ----------------------- |
| movb    | Move byte               |
| movw    | Move word               |
| movl    | Move double word        |
| movq    | Move quad word          |
| movabsq | Move absolute quad word |

指令的格式如下：
$$
movq~~Source,~Destination
$$
其中的Source和Destination可以有如下搭配：

| 移出来源 | 移入目标 | 示例指令          | C语言中的等价表达式 |
| -------- | -------- | ----------------- | ------------------- |
| $Imm$    | $Reg$    | movq $0x4,%rax    | tmp = 0x4;          |
| $Imm$    | $Mem$    | movq $-147,(%rax) | *p = -147;          |
| $Reg$    | $Reg$    | movq %rax,%rdx    | temp2 = temp1;      |
| $Reg$    | $Mem$    | movq %rax,(%rdx)  | *p = tmp;           |
| $Mem$    | $Reg$    | movq (%rax),%rdx  | tmp = *p;           |

简单概括即为：

- 不能将常数$Imm$作为移入的目标
- $Mem$不能同时为移出的源头与移入的目标，要想实现该操作需要进行两步：
  - 将目标值存入寄存器
  - 从寄存器中取出该值移入内存
- 当移出和移入的均为寄存器时，两个寄存器应保持大小相同（见练习3.3）



指令的后缀需要同时以移出和移入的目标为依据进行判断，由于内存的大小不固定，故通常是根据寄存器大小进行判断（见练习3.2）。

以上所述的数据转移是在移出移入的数据类型相同的前提下进行的，但更多时候会遇到的是不同的数据类型转换，需要进行扩展或截断，而截断不需要通过指令进行，故需要进行扩展的指令介绍。

##### 需要进行类型转换的数据转移

零扩展：

| 指令   | 对应含义                       |
| ------ | ------------------------------ |
| movzbw | 在byte前加0使其成为word        |
| movzbl | 在byte前加0使其成为double word |
| movzwl | 在word前加0使其成为double word |
| movzbq | 在byte前加0使其成为quad word   |
| movzwq | 在word前加0使其成为quad word   |

符号扩展：

| 指令   | 对应含义                                   |
| ------ | ------------------------------------------ |
| movsbw | 在byte前加最高位值使其成为word             |
| movsbl | 在byte前加最高位值使其成为double word      |
| movswl | 在word前加最高位值使其成为double word      |
| movsbq | 在byte前加最高位值使其成为quad word        |
| movswq | 在word前加最高位值使其成为quad word        |
| movslq | 在double word前加最高位值使其成为quad word |
| cltq   | 将%eax符号扩展为%rax                       |

**在C语言标准中，若同时需要进行符号转换和大小转换，需要先保证大小变化。**例如$char$转化为$unsigned$时，需要进行的是符号扩展。（见练习3.4）



#### 数据的入栈和出栈

在程序运行中，若用于保存变量的六个寄存器（%rdi, %rsi, %rdx, %rcx, %r8, %r9)均已存放数据，即程序用到六个以上的变量时，便会需要将变量存入**栈**中。**只有当需要时，才会向内存申请栈空间用于存放数据。**

这里的栈与数据结构中的栈的性质相同，被保存在内存的某一部分。与数据结构中的栈不同的是，这里的栈是向下开辟空间的。栈底的地址最大，栈顶的地址最小，且随着数据入栈，栈顶的地址将逐渐减小。寄存器中的%rsp用于记录栈顶的地址。

用指令$pushq$与$popq$进行数据的入栈与出栈。

##### 数据入栈

数据入栈的指令为$pushq~Src$，此时需要先将栈顶指针向下移动8个$Byte$以开辟新的栈空间，随后将数据存入栈中。

假设数据存放在%rbp中，那么以下代码与$pushq~Src$等价：

```
subq $8, %rsp		减少栈顶指针的值
movq %rbp, (%rsp)		将数据入栈
```

但是$pushq$指令只需要一个字节，而上述指令则需要八个字节。

##### 数据出栈

数据出栈的指令为$popq~Src$，与入栈时操作顺序相反，首先需要根据%rsp所指的地址读取存放在该处的数据，随后再将%rsp增加（上移）8个字节。

以下代码与$popq~Src$等价：

```
movq (%rsp), %rax	读取栈顶指针所指的数据
addq $8, %rsp		将栈顶指针向栈底移动
```

**需要注意，此时存放在原来%rsp所指的地址的数值依然保留，直到它下一次被数据入栈的操作覆盖才会消失，改变的只有%rsp所指的地址。**

