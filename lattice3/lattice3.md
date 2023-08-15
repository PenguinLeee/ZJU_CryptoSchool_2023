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

