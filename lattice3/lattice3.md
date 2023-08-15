# Lattice 3-4: LWE for PKE\IBE\FHE, ideal lattice and RLWE

在这一篇文章中，我们将涉及：
1. $LWE \leq PKE$，此即，我们可以基于LWE构造公钥加密
2. 进一步地，我们如何构造签名？IBE？FHE？怎么优化参数？
3. 如何基于理想格/RLWE做一些事情？

此即：构造
$$LWE \leq SIG$$
$$LWE \leq IBE$$
$$LWE \leq FHE$$
（在后两者中，我们将使用Trapdoor技巧）

------

首先，我们回顾Oded Regev的PKE构造方案。它是很多IBE/FHE方案的基础。

$\operatorname{KeyGen}$：我们生成$A \in \mathbb Z_q^{n \times m}$。（这里需要$A$是一个矮胖的矩阵，即$n < m$）。$r$是一个短向量，不妨认为$r \in \{0, 1\}^m$。计算$u = Ar$。
于是公钥$pk = (A, u = Ar + e)$，私钥$sk = r$。

$\operatorname{Enc}_{pk}(m)$：我们生成均匀随机(uniformly random)的$s \leftarrow \mathbb{Z}_q^n$，计算$(c_1, c_2) = (sA+e, su+e'+\frac{q}{2}m)$。则$(c_1, c_2)$为密文。

$\operatorname{Dec}_{pk}(c_1, c_2)$：计算$\lfloor c_2 - c_1 \cdot r\rceil$。

我们考察解密正确性，即计算$c_2 - c_1 \cdot r$。

$$c_2 - c_1r = su+e'+\frac{q}{2}m - (sA+e)r = sAr+se +e'- sAr - er = \frac{q}{2}m + e' + se - er$$



