# Hash

原文： https://jorgechavez.dev/2020/11/12/string-hashing/
作者：Jorge Chávez


## 哈希公式

```math
hash(s) = s[0] + s[1]P + s[2]P^2 + \dots + s[n-1]P^{n-1} \mod N
```
P：是一个质数。 如果我们要计算的哈希只包括小写英文字母的字符的一个字，31将是一个好数字。但是，如果我们将包括大写字符，那么53是一个合适的选择。

N：这是一个大的质数。 比如：$10^9 + 9$

python 代码：
```py
P = 31
N = 10 ** 9 + 9

def hash(s):
    h = 0
    for i in range(0, len(s)):
        h += ord(s[i]) * pow(P, (i))
    print(h)
    print(h % N)

if __name__ == '__main__':
    hash('apple')
```

