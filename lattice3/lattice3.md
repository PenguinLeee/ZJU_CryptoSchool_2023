# Lattice 3-4: LWE for PKE\IBE\FHE, ideal lattice and RLWE

在这一篇文章中，我们将涉及：
1. $LWE \leq PKE$，此即，我们可以基于LWE构造公钥加密
2. 进一步地，我们如何构造签名？IBE？FHE？怎么优化参数？
3. 如何基于理想格/RLWE做一些事情？

于是公钥$pk = (A, u = Ar + e)$，私钥$sk = r$。

$\operatorname{Enc}_{pk}(m)$：我们生成均匀随机(uniformly random)的$s \leftarrow \mathbb{Z}_q^n$，计算$(c_1, c_2) = (sA+e, su+e'+\frac{q}{2}m)$。则$(c_1, c_2)$为密文。

$\operatorname{Dec}_{pk}(c_1, c_2)$：计算$\lfloor c_2 - c_1 \cdot r\rceil$。

我们考察解密正确性，即计算$c_2 - c_1 \cdot r$。

$$c_2 - c_1r = su+e'+\frac{q}{2}m - (sA+e)r = sAr+se +e'- sAr - er = \frac{q}{2}m + e' + se - er$$



