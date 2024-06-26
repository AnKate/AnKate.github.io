---
title: CSAPP 2.3&2.4
published: 2020-09-26
description: 整型数据运算、浮点数
image: ''
tags: [CSAPP]
category: Notes
draft: false 
---



## 整型数据运算

### 无符号加法

对于两个由$w$位二进制数组成的非负数字$x$和$y$，$0 \leq x,y \leq  2^w - 1$ 。

当我们计算这样两个无符号数之和时， $0\leq x + y \leq 2^{w+1}-2$，出现了需要用$w+1$位二进制数表示的情况。

这时的解决方法是截掉最高位，变回$w$位二进制数，相当于在结果数值上减去$ 2^w $，即如下情况：
$$
x +^u_wy = 
	\begin{cases}
	x + y, & x+y < 2^w\\
	x + y - 2^w, &2^w \leq x + y < 2^{w+1}
	\end{cases}
$$


- 无符号数加法是否溢出：判断$x+y$是否小于$x$和$y$，若是，则溢出。

- 无符号数取反：
  $$
  -^u_wx =
  	\begin{cases}
  	x, & x=0\\
  	2^w - x, & x>0
  	\end{cases}
  $$

### 补码加法

与无符号加法原理类似，对于两个$w$位二进制数补码组成的的$x,y$，$ -2^{w-1} \leq x,y \leq 2^{w-1}-1 $。

两个补码之和的范围则有$-2^w \leq x+y \leq 2^w-2$，变成了需要由$w+1$位二进制数表示的情况。

同样地，截掉最高位，但由于补码最高位具有的性质，导致转换成十进制数值时出现了三种可能性：
$$
x+^t_wy =
	\begin{cases}
	x+y-2^w, &2^{w-1}\leq x+y\\
	x+y, &-2^{w-1}\leq x+y < 2^{w-1}\\
	x+y+2^w, &x+y <-2^{w-1}
	\end{cases}
$$

- 判断补码加法的溢出：将$x+y$与$x,y$分别与$0$进行比较，$x,y <0$且$x+y\ge 0$，或$x,y >0$且$x+y < 0 $时溢出，否则无溢出。

- 补码取反：

$$
-^t_wx=
	\begin{cases}
	TMin_w, &x=TMin_w\\
	-x, &x>Tmin_w
	\end{cases}
$$

- 十六进制数取反：

  首先将十六进制数转化为十进制数，因为取反判断依据为十进制数值与$Tmin_w$的大小关系。判断十六进制的数值是否超出$TMax_w$，若溢出，则转化为二进制数后截掉最高位，后转化为十进制数值；若未溢出，则直接转化为对应数值。

  随后再按照补码取反的规则进行取反。

  最后将十进制数重新转化为十六进制数，正数保持不变；若为负数，首先求该负数的相反数的原码，再取所求原码的补码，最后加上一，所得的二进制数按照无符号数解读，则为最终的十六进制数。

  详情见书P95练习2.23.

### 无符号乘法

对于两个由$w$位二进制数组成的非负数字$x$和$y$，$0 \leq x,y \leq  2^w - 1$ 。

这样两个数的乘积的范围有$0\leq x*y\leq(2^w-1)^2$，需要用$2w$位的二进制数进行表示。

处理方法是对$2^w$取模，即截断为$w$位。
$$
x*^u_wy = (x*y) \mod {2^w}
$$


### 补码乘法

对于两个$w$位二进制数补码组成的$x,y$，$-2^{w-1} \leq x,y \leq 2^{w-1}-1$。

这样两个数的乘积的范围有$-2^{2w-2}+2^{w-1} \leq x*y \leq 2^{2w-2}$，需要用$2w$位的二进制数进行表示。

处理方法与无符号乘法类似，截断为$w$位后从无符号转化为补码。
$$
x*^t_wy=U2T_w((x*y)\mod2^w)
$$


### 和常数的乘法

首先，某个整数$x$乘以$2^k$可以等价将$x$的二进制数左移$k$位，并将该$x$按照对应编码的解读方式进行运算（即按照补码或是无符号方法进行解读）。

又已知任意一个十进制数可以由二进制数进行表示，故在乘以其他整数时，可以转化为若干次左移运算的求和：
$$
N=(2^{a_1}+2^{a_2}+…+2^{a_m})\\
x*N=x*(2^{a_1}+2^{a_2}+…+2^{a_m})=(x<<a_1)+(x<<a_2)+…+(x<<a_m)
$$


例如：
$$
x*11=x*(2^3+2^1+2^0)=(x<<3)+(x<<1)+(x<<0)
$$


### 除以2的整数次幂

首先注意，除以$2^k$不完全等同于右移$k$位，尽管在无符号数的案例中的效果是相同的，但是在处理补码中的负值的除法时，两者有一些舍入上的微妙差异。

- 右移$k$位的舍入方法总是为向下舍入，即数值为正时去除小数点部分，数值为负时去除小数点部分并且减去一，即：
  $$
  x>>k=\lfloor x/2^k\rfloor
  $$

- 补码的负数在除以$2^k$后的舍入情况为向0舍入，即舍入方向均为向0靠近：
  $$
  x/2^k=
  	\begin{cases}
  	\lfloor x/2^k \rfloor, &x\geq0\\
  	\lceil x/2^k \rceil, & x<0
  	\end{cases}
  $$
  在处理负值的舍入时，并不能单纯视作右移$k$位，需要进行向上舍入的操作。换言之，除法与右移k位的等价关系即为：
  $$
  x/2^k=
  	\begin{cases}
  	x>>k,&x\geq0\\
  	(x+(1<<k)-1)>>k,&x<0
  	\end{cases}
  $$



## 浮点数

### 小数的二进制表示

对于任意整数$b_mb_{m-1}…b_2b_1b_0.b_{-1}b_{-2}…b_{-n+1}b_{-n}$，整数部分的二进制数$b_mb_{m-1}…b_2b_1b_0$等同于$\sum2^i * b_i$，与之类似地，小数部分的二进制数$b_{-1}b_{-2}…b_{-n+1}b_{-n}$，代表着$\sum2^i*b_i$。

与十进制中无法用有限位小数$\frac{1}{3}$类似，二进制中只能用有限位小数表示那些能写成$x*2^y$的数字，而那些不能被表示成这种形式的小数，则需要用类似的循环小数进行表示，如表示$\frac{1}{5}$：





| 二进制表示形式 | 对应分数数值     | 对应十进制数值    |
| -------------- | ---------------- | ----------------- |
| $0.0_2$        | $\frac{0}{2}$    | $0.0_{10}$        |
| $0.01_2$       | $\frac {1}{4}$   | $0.25_{10}$       |
| $0.010_2$      | $\frac{2}{8}$    | $0.25_{10}$       |
| $0.0011_2$     | $\frac{3}{16}$   | $0.1875_{10}$     |
| $0.00110_2$    | $\frac{6}{32}$   | $0.1875_{10}$     |
| $0.001101_2$   | $\frac{13}{64}$  | $0.203125_{10}$   |
| $0.0011010_2$  | $\frac{26}{128}$ | $0.203125_{10}$   |
| $0.00110011_2$ | $\frac{51}{256}$ | $0.19921875_{10}$ |





### IEEE浮点数表示规则

与十进制数的科学计数法类似，在二进制中，可以将V以2为基数进行科学计数法的表示：
$$
V=(-1)^s*M*2^E
$$
在公式中：

- $S$是符号位，在二进制表示中占第一位，$S=0$时代表正数，$s=-1$时代表负数。在二进制表示中占首位。
- $M$是尾数，在$32$位的二进制表示中占$23$位，$64$位的二进制表示中占$52$位。由$frac=f_{n-1}…f_1f_0$表示，但并不完全等同于$M$。在不同的情况下取值范围有$(0,1-\epsilon]$与$[1,2-\epsilon)$两种情况。
- $E$为指数，$exp=e_{k-1}…e_1e_0$是$E$的编码，在$32$位中占$8$位，$64$位中占$11$位，但$exp$不等同于指数，需要通过偏置操作后才能转化为$E$。

在存储方式上，尾数$M$排在$exp$的后面。目的是为了便于进行比较浮点数的大小，通过直接比较指数的不同便能较快的得出大小关系。

浮点数分为三种。

#### Normalized Values

这一类的浮点数的$exp$不全为1或不全为0，而$E=exp-Bias$，其中的$Bias=2^{k-1}-1$，为一定值。

规定$M=1+frac，M\in[1,2)$。由于保留的整数部分只能为1，故可以省略整数部分的1位，用于增加小数点部分的一位精度。

#### Denormalized Values

这一类的浮点数的$exp=0$，规定此时$E=1-Bias，M=f$，与前一类有所差别。

这一类浮点数的定义有两类用途，当$frac$部分全部取0时，可以用来表示$0.0$，特别地，$S=1$，其余为全取0时，能够获得$-0.0$，这两者在有些方面有所差别，但在其他方面类似（书上未展开说明）；$frac$部分不全为0时，还能用来表示非常接近$0.0$的数。

#### Special Values

第三类浮点数的$exp$每一位都取1，用于表示一些特殊定义。

$S$与$frac$全部取0时，可以用来表示$\infty$，$S=1$而$frac$全部取0时，用于表示$-\infty$。特别地，$1.0/0.0=\infty，1.0/-0.0=-\infty$。

$frac$不全为0时，代表$NaN$，在一些没有定义的计算中会输出这一结果，如$\sqrt{-1}、\infty-\infty$。

### 浮点数的舍入

浮点数的舍入通常情况下为**向偶舍入**，在该数小于中值时向下舍入，大于中值时向上舍入，等于中值时向最近的偶数舍入。

二进制数字的舍入情况（保留一位小数）：





| 舍入前     | 舍入后   | 舍入情况（向偶舍入） |
| ---------- | -------- | -------------------- |
| $10.010_2$ | $10.0_2$ | 向下舍入             |
| $10.011_2$ | $10.1_2$ | 向上舍入             |
| $10.110_2$ | $11.0_2$ | 向上舍入             |
| $11.001_2$ | $11.0_2$ | 向下舍入             |





简言之，保留位数的后面若为$10…0$（最高位为1，其余为0）的形式，则需要让保留后的最低有效位数为0；若除去最高位外仍有若干个1，则向上舍入；最高位不为1，则向下舍入。

- 按照十进制数值转化为IEEE浮点数表达形式的方法：

  首先将十进制小数写成二进制表达式，后表达为二进制的科学计数法，即形如$M*2^E$的式子。

  根据题目给定的$frac$部分位数进行判断，若超出给定位数则进行舍入，不超出则保留，最终结果即为$frac$部分的二进制数。

  根据$exp=E+Bias$，可求出$exp$部分的值，按照二进制代码展开，即为$exp$部分的二进制数。

  详情见书P122 练习2.52.

### 浮点数运算

#### 加法

对于两个浮点数，其加法为：
$$
(-1)^s*M*2^E=(-1)^{s_1}*M_1*2^{E_1}+(-1)^{s_2}*M_2*2^{E_2}
$$
其中的$s,M$需要对齐小数点，再分别做加法，$E$即为对齐后的$E_1$。

加法性质：

- 封闭（包括$NaN$）：集合中任意两个元素$a,b$通过运算$f$得出的结果仍在集合中
- 满足加法交换律
- **加法结合律不成立**
- 0的性质成立：任意一个数加上0仍然为其本身
- 相反数存在（$\infty$和$NaN$除外）
- 单调性成立（$\infty$和$NaN$除外）：$a\geq b \Rightarrow a+c \geq b+c$

#### 乘法

其乘法为：
$$
(-1)^s*M*2^E=(-1)^{s_1}*M_1*2^{E_1}*(-1)^{s_2}*M_2*2^{E_2}
$$
其中$s=s_1+s_2，M=M_1*M_2，E=E_1+E_2$。

乘法性质：

- 封闭
- 乘法交换律成立
- **乘法结合律不成立**
- 1的性质成立：任意一个书乘以1仍然为其本身
- **乘法分配律不成立**：$a*(b+c)\neq a*b+b*c$
- 单调性成立（$\infty$和$NaN$除外）：$a\geq b \bigcup c\geq0 \Rightarrow a*c\geq b*c$
- 若运算后的$M\geq2$，则将其右移并且增加$E$。
- 若$E$超过范围，则溢出。
- 若$E$未超过范围，则舍入$M$，计算得到$frac$。

#### 注意事项

运算后若$M\geq2$，则右移并增加$E$。

若$E$超过范围，则溢出；若未超过，则舍入$M$，并计算得出$frac$。

关于溢出：

- 若E超过上界，则上溢(Overflow)，表示为$\infty$
- 若E低于下界，则下溢(Underflow)，显示为0

### C语言中的浮点数

C语言中，浮点数的数据类型有$float$和$double$两种，它们与整型$int$的类型转换会改变位表示。

$double/float\rightarrow int$：类似于向0舍入（超范围未定义，通常设为$TMin$）

$int \rightarrow double$：由于$float$的整数部分与小数部分能够表示的范围更大，故能够精准转换

$int \rightarrow float$：数字不会溢出，但是会根据规则舍入
