* * *

**新的算法并没有透露 `n`，只给定了两个大整数：`(p*q)^(p+q)` 和 `(p*q)^(p-q)`，其中 `^` 是按位异或运算。**


```python
import sympy

p = sympy.randprime(2 ** 1023, 2 ** 1024)
q = sympy.randprime(2 ** 1023, 2 ** 1024)

a = (p * q) ^ (p + q)
b = (p * q) ^ (p - q)

flag = open('flag.txt', 'rb').read()
m = int.from_bytes(flag, 'big')

print(a, b, pow(m, 65537, p * q))

```



* * *

## 2018年12月20日更新

我们定义 $ f_1(x,y) = (x \ast y)^{(x+y)} 和 f_2(x,y) = (x \ast y)^{(x-y)} $ ，我们发现这两个函数都有一个共同的性质，就是函数值的最低 n 个二进制位只和 x、y 的最低 n 个二进制位有关。也就是说，我们可以用 a 和 b 的最低 n 位来判断 p 和 q 的最低 n 位是否可能正确。如果它们的最低 n 位满足 $ f_1 $和 $f_2$ 函数，那么它们就是 p 和 q 低位的候选答案；如果不满足，它们就根本不可能是真正 p 和 q 的低位。所以我们可以从一个二进制位（n=1）开始，每次增加一位。每增加一位时，我们把原来满足条件的 p 和 q 低位的每种可能情况分别在前面加上 0 或 1，这样每种情况就变成了 4 种新的情况，然后对所有新的情况用 $ f_1 $ 和 $ f_2 $ 函数提供的约束条件进行过滤，只保留满足条件的情况。当跑到 1024 位的时候，就只会剩下真正满足条件的 p 和 q 了。然后，我们根据 RSA 的原理，在 mod (p-1)\*(q-1) 的意义下对 e 求逆元，得到私钥 d，计算 pow(c, d, p\*q)即可得到 flag 的大整数表示。

**求解脚本如下**

```python
import gmpy2

a, b, c = [int(s) for s in open('output.txt').read().split()]
#假设已将加密内容保存到 output.txt 文件中

f1 = lambda p, q: (p * q) ^ (p + q)
f2 = lambda p, q: (p * q) ^ (p - q)

candidates = {(0, 0)}

for m in range(1025):
    print(m, len(candidates))
    candidates_ = set()
    mask = (2 << m) - 1
    for x, y in candidates:
        if f1(x, y) == a and f2(x, y) == b:
            p, q = x, y
            d = gmpy2.invert(65537, (p - 1) * (q - 1))
            m = pow(c, d, p * q)
            print(bytes.fromhex(hex(m)[2:]))
            exit()
        for bx in range(2):
            for by in range(2):
                xx = x + (bx << m)
                yy = y + (by << m)
                if f1(xx, yy) & mask != a & mask:
                    continue
                if f2(xx, yy) & mask != b & mask:
                    continue
                candidates_.add((xx, yy))
candidates = candidates_

```

**有几个人做出来了呢（坏笑:）**

[^1]hackergame2018-RSA_of_Z