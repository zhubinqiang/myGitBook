# 计算机图形学入门
[TOC]

以 闫令琪 GAMES101-现代计算机图形学入门 https://www.bilibili.com/video/BV1X7411F744 的视频做的笔记


这段话把英文26个字母都包含进去了
> the quick brown fox jumps over the lazy dog

## 向量
在图形学中 默认是列向量

$\vec{A} = \begin{pmatrix} x \\ y \end{pmatrix}$

$A^T = (x, y)$

$|\vec{A}| = \sqrt{x^2 + y^2}$

$\overrightarrow{AB} = (x_{b}-x_{a}, y_{b}-y_{a})$

单位向量 
$$ \hat{n} = \frac{\vec{u}}{|\vec{u}|} $$

### 向量的点乘
向量的点乘是一个数

$$ \vec{A} \cdot \vec{B} = |\vec{A}| |\vec{B}| \cos\theta $$

$$\vec{A} = (x_{a}, y_{a})$$ 

$$\vec{B} = (x_{b}, y_{b})$$

$$ \vec{A} \cdot \vec{B} = x_{a}x_{b} + y_{a}y_{b} $$

其中 $0 \le \theta \le \pi$


这个公式能快速知道2个向量之间的夹角
$$ \cos\theta = \frac{\vec{A} \cdot \vec{B}}{|\vec{A}| |\vec{B}|}  $$

### 点乘的运算法则
交换律
$\vec{A} \cdot \vec{B} = \vec{B} \cdot \vec{A}$

结合律
$\vec{A} \cdot (\vec{B} + \vec{C}) = \vec{A} \cdot \vec{B} + \vec{A} \cdot \vec{C}$

分配律
$(k\vec{A}) \cdot \vec{B} = \vec{A} \cdot (k\vec{B}) = k(\vec{A} \cdot \vec{B})$

### 向量的叉乘
![](images/introduction-to-computer-graphics/330px-Cross_product_vector.svg.png)

向量叉乘的结果为一个向量，有大小有方向.
方向: 用右手螺旋定则判断. 先四指方向指向 $\vec{a}$ 的方向, 再以不超过180°转向 $\vec{b}$. 大拇指方向就是  $ \vec{a} \times \vec{b} $ 的方向

$$ \vec{a} \times \vec{b} = |\vec{a}| |\vec{b}| \sin\theta $$

$$ \vec{a} \times \vec{b} = 
\begin{vmatrix} y_{a}z_{b} - y_{b}z_{a} \\
z_{a}x_{b} - x_{z}z_{b} \\
x_{a}y_{b} - y_{a}x_{b}
\end{vmatrix}
$$

$$
\vec{a} \cdot \vec{b} =
A*b =
\begin{pmatrix}
0 & -z_{a} & y_a \\
z_{a} & 0 & -x_a \\
-y_{a} & x_{a} & 0
\end{pmatrix}
\begin{pmatrix}
x_b \\
y_b \\
z_b
\end{pmatrix}
$$

### 叉乘的运算法则
没有交换律
$\vec{x} \times \vec{y} = \vec{z}$

$\vec{A} \times \vec{B} = -\vec{B} \times \vec{A}$

$\vec{A} \times \vec{A} = \vec{0}$

$\vec{A} \times ( \vec{B} + \vec{C} )= \vec{A} \times \vec{B} + \vec{A} \times \vec{C} $

$\vec{A} \times (k\vec{B}) = k(\vec{A} \times \vec{B})$

### 分解
$|\vec{u}| = |\vec{v}| = |\vec{w}| = 1$

$\vec{u} \cdot \vec{v} = \vec{u} \cdot \vec{w} = \vec{v} \cdot \vec{w} = 0$

$\vec{w} = \vec{u} \cdot \vec{v}$ (右手定则)

$\vec{p} = (\vec{p} \cdot \vec{u}) \vec{u} + (\vec{p} \cdot \vec{v}) \vec{v} +(\vec{p} \cdot \vec{w}) \vec{w}$



## 矩阵
### 矩阵乘法
矩阵乘法 第1个矩阵的列数 = 第2个矩阵的行数 才有意义
(M x N) (N x P) = (M x P)

$$
\begin{bmatrix} 1 & 3 \\
5 & 2 \\
0 & 4
\end{bmatrix}
\begin{bmatrix} 3 & 6 & 9 & 4 \\ 
2 & 7 & 8 & 3
\end{bmatrix} =
\begin{bmatrix} 9 & ? & 33 & 13 \\
19 & 44 & 61 & 26 \\
8 & 26 & 32 & 12
\end{bmatrix}
$$

这里的 ？是 第1行第2列， 
然后去第1个矩阵里面找第1行 $\begin{bmatrix} 1 & 3\end{bmatrix}$

以及第2个矩阵里面找第2列  $\begin{bmatrix} 6 \\ 7\end{bmatrix}$

然后运算 1 * 6 + 3 * 7 = 27

### 矩阵的运算法则
没有结合律 通常情况下 $AB \neq BA $

有分配律
1. (AB)C = A(BC)
2. A(B+C) = AB + AC
3. (A+B)C = AC + BC

### 矩阵转置
矩阵转置就是把行列互换
$$
\begin{bmatrix}
1 & 2 \\
3 & 4 \\
5 & 6
\end{bmatrix}^T =
\begin{bmatrix}
1 & 3 & 5 \\
2 & 4 & 6
\end{bmatrix}
$$

$$(AB)^T = B^TA^T$$

### 单位矩阵
单位矩阵: 左上到右下这条对角线上面都是1，其余都是0
$$
I_{3\times3} =
\begin{pmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{pmatrix}
$$

矩阵的逆
$ AA^{-1} = A^{-1}A = I $
$ (AB)^{-1} = B^{-1}A^{-1} $

### 矩阵中向量的点乘
$$
\vec{A} \cdot \vec{B} =
\vec{A}^{T} \vec{B} =
\begin{pmatrix}
x_a & y_a & z_a
\end{pmatrix}
\begin{pmatrix}
x_b \\
y_b \\
z_b
\end{pmatrix} =
(x_ax_b + y_ay_b + z_az_b)
$$

### 矩阵中向量的叉乘
$$
\vec{A} \cdot \vec{B} =
A*b =
\begin{pmatrix}
0 & -z_{a} & y_a \\
z_{a} & 0 & -x_a \\
-y_{a} & x_{a} & 0
\end{pmatrix}
\begin{pmatrix}
x_b \\
y_b \\
z_b
\end{pmatrix}
$$

### 2D线性变换
$$
\begin{bmatrix} a_{11} & a_{12} \\
a_{21} & a_{22}
\end{bmatrix}
\begin{bmatrix} x \\ y \end{bmatrix} =
\begin{bmatrix} a_{11}x + a_{12}y \\
a_{21}x + a_{22}y
\end{bmatrix}
$$


## 参考
https://www.zhihu.com/column/c_1249465121615204352




