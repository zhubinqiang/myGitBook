# LATEX数学公式基本语法

转载于 侯凯 [博客园](https://www.cnblogs.com/houkai/p/3399646.html)

[TOC]

参考材料： 
1. http://www.mohu.org/info/symbols/symbols.htm
2. https://en.wikibooks.org/wiki/LaTeX/Mathematics#List_of_mathematical_symbols
3. https://aaron-bird.github.io/2019/09/29/%E5%B8%B8%E7%94%A8%E7%9A%84%20LaTeX%20%E7%AC%A6%E5%8F%B7%E5%85%AC%E5%BC%8F/
4. https://oeis.org/wiki/List_of_LaTeX_mathematical_symbols


TEX 是Donald E. Knuth 编写的一个以排版文章及数学公式为目标的计算机程序。TEX的版本号不断趋近于π，现在为3.141592。由Pascal 语言写成，特点: 免费、输出质量高、擅长科技排版、有点像编程。

LATEX 目前使用最广泛的TEX 宏集。 每一个LATEX 命令实际上最后都会被转换解释成几个甚至上百个TEX命令。

CTEX 国内致力于TEX 推广的网站：http://www.ctex.org。该网站提供了CTEX 中文套装，这个安装程序把MiKTEX（TEX 在Windows 操作系统上的实现版本）和一些相关工具（如WinEdt、GSview 等）打包在一起，同时对中文接口进行了配置，以实现对中文文本的编辑。

如果想学习LATEX安装CTEX套装就可以了。LATEX 的功能和宏包有很多，每个人用到的功能是有限的；边用边学，建立了基本的概念以后，在使用中根据需求去解决问题就可以了。本文主要简单介绍LATEX的数学排版。

## 基础知识
1.LATEX控制序列的概念（类似于函数）

控制序列可以是作为命令：以“\”开头，参数：必须参数{}和可选参数[]。

2.环境概念 
以“bengin 环境名”开始，并以“end 环境名”结束。

3.LATEX可以排版公式与文字，故分为：数学模式和文本模式。如果你想要在公式中排版普通的文本（直立字体和普通字距），那么你必须要把这些文本放在\textrm{...} 命令中。

4.在数学模式中又分为两种，一种是公式排版在一个段落之中；另一种是公式独立形式排版。前一种，公式直接放在文字之间，公式高度一般受文本高度限制；后一种，公式另起一行，高度可调整。处于段内的数学文本要放在`\\(`与`\\)` 之间， $ 与$ 之间，或者`\\begin{math}` 与`\\end{math}` 之间；处于段外的数学文本放在`\\[` 与`\\]` 之间， `$$` 与 `$$` 之间，或者`\\begin{displaymath}` 与`\\end{displaymath}` 之间（为了网页显示，这里用双斜杠表示单斜杠）。如下：

```
\sum_{i=0}^{n}i^2
```

```math
\sum_{i=0}^{n}i^2
```

## 数学公式基本语法
### 1. 上标与下标
上标命令是 ^{角标}，下标命令是 _{角标}。当角标是单个字符时可以不用花括号（在 LaTeX 中，花括号是用于分组，即花括号内部文本为一组）。
```
x_1

x_1^2

x^2_1

x_{22}^{(n)}

{}^*\!x^*
```

```math
x_1

x_1^2

x^2_1

x_{22}^{(n)}

{}^*\!x^*
```

### 2. 分式
输入较短的分式时，最简单的方法是使用斜线，譬如输入`(x+y)/2`，可得到`(x+y)/2`。

要输入带有水平分数线的公式，可用命令：`\frac{分子}{分母}`。
```
\frac{x+y}{2}

\frac{1}{1+\frac{1}{2}}
```

```math
\frac{x+y}{2}

\frac{1}{1+\frac{1}{2}}
```

### 3. 根式
排版根式的命令是：开平方：`\sqrt{表达式}`；开n次方：`\sqrt[n]{表达式}`
```
\sqrt{2}<\sqrt[3]{3}

\sqrt{1+\sqrt[p]{1+a^2}}

\sqrt{1+\sqrt[^p\!]{1+a^2}}
```

```math
\sqrt{2}<\sqrt[3]{3}

\sqrt{1+\sqrt[p]{1+a^2}}

\sqrt{1+\sqrt[^p\!]{1+a^2}}
```

### 4. 求和与积分
排版求和符号与积分符号的命令分别为 `\sum` 和 `\int`，它们通常都有上下限，在排版上就是上标和下标。
```
\sum_{k=1}^{n}\frac{1}{k}

\sum_{k=1}^n\frac{1}{k}

\int_a^b f(x)dx

\int_a^b f(x)dx

\int_a^b f(x)\mathrm{d}x
```

```math
\sum_{k=1}^{n}\frac{1}{k}

\sum_{k=1}^n\frac{1}{k}

\int_a^b f(x)dx

\int_a^b f(x)dx

\int_a^b f(x)\mathrm{d}x
```

### 5. 公式中的空格
LaTeX 能够自动处理公式中的大多数字符之间的空格，但是有时候需要自己手动进行控制。

紧贴: `a\!b`
```math
a\!b
```

没有空格: `ab`
```math
ab
```

小空格: `a\,b`
```math
a\,b
```

中等空格: `a\;b`
```math
a\;b
```

大空格: `a\ b`
```math
a\ b
```

quad空格: `a\quad b`
```math
a\quad b
```

两个quad空格: `a\qquad b`
```math
a\qquad b
```

换行 `\newline`
```math
a
\newline
b
```

在公式中灵活的运用空格命令可以起到美化公式的作用
```math
\int_a^b f(x)\mathrm{d}x

\int_a^b f(x)\,\mathrm{d}x
```


### 6. 公式中的定界符
这里所谓的定界符是指包围或分割公式的一些符号
```
\left(\sum_{k=\frac{1}{2}}^{N^2}\frac{1}{k}\right)
```

```math
\left(\sum_{k=\frac{1}{2}}^{N^2}\frac{1}{k}\right)
```

### 7. 矩阵
对于少于 10 列的矩阵，可使用 matrix，pmatrix，bmatrix，Bmatrix，vmatrix 和 Vmatrix 等环境。
```
\begin{matrix}1 & 2\\3 &4\end{matrix}

\begin{pmatrix}1 & 2\\3 &4\end{pmatrix}

\begin{bmatrix}1 & 2\\3 &4\end{bmatrix}

\begin{Bmatrix}1 & 2\\3 &4\end{Bmatrix}

\begin{vmatrix}1 & 2\\3 &4\end{vmatrix}

\begin{Vmatrix}1 & 2\\3 &4\end{Vmatrix}
```

```math
\begin{matrix}1 & 2\\3 &4\end{matrix}

\begin{pmatrix}1 & 2\\3 &4\end{pmatrix}

\begin{bmatrix}1 & 2\\3 &4\end{bmatrix}

\begin{Bmatrix}1 & 2\\3 &4\end{Bmatrix}

\begin{vmatrix}1 & 2\\3 &4\end{vmatrix}

\begin{Vmatrix}1 & 2\\3 &4\end{Vmatrix}
```

> & 表示对齐位置,同一行可以有多个 &

### 8. 排版数组
当矩阵规模超过 10 列，或者上述矩阵类型不敷需求，可使用 array 环境。该环境可把一些元素排列成横竖都对齐的矩形阵列。
```
\mathbf{X} =
\left( \begin{array}{ccc}
x_{11} & x_{12} & \ldots \\
x_{21} & x_{22} & \ldots \\
\vdots & \vdots & \ddots
\end{array} \right)
```

```math
\mathbf{X} =
\left( \begin{array}{ccc}
x_{11} & x_{12} & \ldots \\
x_{21} & x_{22} & \ldots \\
\vdots & \vdots & \ddots
\end{array} \right)
```

```math

A_{m,n} = 
 \begin{pmatrix}
  a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\
  a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\
  \vdots  & \vdots  & \ddots & \vdots  \\
  a_{m,1} & a_{m,2} & \cdots & a_{m,n} 
 \end{pmatrix}
```

## 运算符符号
$ 1 \cdot 2 $
$ 1 \times 2$
$ 1 \div 2$
$ 1 \lt 2 $
$ 1 \le 2 $
$ 2 \gt 1 $
$ 2 \ge 1 $
$ 2 \neq 1 $
$ \pi \approx 3.14 $
$ A \subset B $
$ A \not\subset B $
$ A \supset B $

## 希腊字母
```math
\alpha, \beta, \gamma, \theta,

\Gamma, \pi, \Pi, 

\phi, \varphi, \mu, \Phi
```


## 例子
### 操作符
```math
\cos (2\theta) = \cos^2 \theta - \sin^2 \theta

\lim\limits_{x \to \infty} \exp(-x) = 0

a \bmod b

x \equiv a \pmod{b}
```

### 指数
```math
k_{n+1} = n^2 + k_n^2 - k_{n-1}

n^{22}

f(n) = n^5 + 4n^2 + 2 |_{n=17}
```

### 分数与二项式
```math
\frac{n!}{k!(n-k)!} = \binom{n}{k}

\frac{\frac{1}{x}+\frac{1}{y}}{y-z}

^3/_7
```

### 求根
```math
\sqrt{\frac{a}{b}}

\sqrt[n]{1+x+x^2+x^3+\dots+x^n}
```


### 自动调整大小
```math
\left(\frac{x^2}{y^3}\right)

P\left(A=2\middle|\frac{A^2}{B}>4\right)

\left\{\frac{x^2}{y^3}\right\}

\left.\frac{x^3}{3}\right|_0^1
```

### 矩阵和数组
```math
M = \begin{bmatrix}
       \frac{5}{6} & \frac{1}{6} & 0           \\[0.3em]
       \frac{5}{6} & 0           & \frac{1}{6} \\[0.3em]
       0           & \frac{5}{6} & \frac{1}{6}
     \end{bmatrix}
```

### 向量
1. 头上戴箭头
```math
\vec x
```

```math
\vec a \quad \overrightarrow{AB}
```

```math
\overrightarrow{AB}
```

```math
\overleftarrow{XY}
```

2. 用黑体表示
```math
\boldsymbol x
```


单位向量
```math
\hat{ab}
```

