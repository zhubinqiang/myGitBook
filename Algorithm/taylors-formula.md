# 泰勒公式

[TOC]

对 知乎问答 [怎样更好地理解并记忆泰勒展开式？](https://www.zhihu.com/question/25627482) 作者：陈二喜 的简单摘录

仿造一段曲线，**要先保证起点相同，再保证在此处导数相同，继续保证在此处的导数的导数相同……**

有一个原函数 $f(x)$，我再造一个图像与原函数图像相似的多项式函数 $g(x)$ ,为了保证相似，我只需要保证这俩函数在某一点的初始值相等，1阶导数相等，2阶导数相等，……n阶导数相等。

首先要在曲线 $f(x)$ 上任选一个点，为了方便，就选 $(0, f(0))$ ,设仿造的曲线的解析式为 $g(x)$ ，前面说了，仿造的曲线是一个多项式，假设算到n阶。

$$
g(x) = a_0 + a_1x + a_2x^2 + a_3x^3 + \ldots + a_nx^n
$$

先保证初始点相同，求出 $a_0$:
$$
g(0) = f(0) = a_0
$$

接下来，必须保证n阶导数依然相等，即
$$
g^{(n)}(0) = f^{(n)}(0)
$$

因为对 $g(x)$ 求n阶导数时，只有最后一项为非零值 $n!a_n$
由此求出: $$
a_n = \frac{f^{(n)}(0)}{n!}
$$

求出了 $a_n$ ，剩下的只需要按照这个规律换数字即可。
综上： 
$$
g(x) = f(0) + \frac{f^{(1)}(0)}{1!}x + \frac{f^{(2)}(0)}{2!}x^2 + \frac{f^{(3)}(0)}{3!}x^3 + \ldots  +\frac{f^{(n)}(0)}{n!}x^n
$$

上式是在 $(0, f(0))$ 处, 但不一定非要从x=0的地方开始，也可以从 $(x_0, f(x_0))$ 开始
此时，只需要将0换成 $x_0$ ，然后再按照上面一模一样的过程重新来一遍

令
$$
g(x) = a_0 + a_1(x - x_0) + a_2(x - x_0)^2 + a_3(x - x_0)^3 + \ldots + a_n(x - x_0)^n
$$

n阶导数相等, 则：
$$
a_n = \frac{f^{(n)}(x_0)}{n!}
$$

当 $x = x_0$ 时，$g(x_0) = a_0 + 0 + 0 + \cdots = f(x_0)$, 则: $a_0 = f(x_0)$

最后就能得到如下结果：
$$
g(x) = f(x_0) + \frac{f^{(1)}(x_0)}{1!}(x-x_0) + \frac{f^{(2)}(x_0)}{2!}(x-x_0)^2 + \frac{f^{(3)}(x_0)}{3!}(x-x_0)^3 + \ldots  +\frac{f^{(n)}(x_0)}{n!}(x-x_0)^n
$$




