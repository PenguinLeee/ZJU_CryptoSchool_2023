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

（最后也没有讲IBE...）

------

首先，我们回顾Oded Regev的PKE构造方案。它是很多IBE/FHE方案的基础。

$\operatorname{KeyGen}$：我们生成$A \in \mathbb Z_q^{n \times m}$。（这里需要$A$是一个矮胖的矩阵，即$n < m$）。$r$是一个短向量，不妨认为$r \in \{0, 1\}^m$。计算$u = Ar$。
于是：
* 公钥$pk = (A, u = Ar + e)$
* 私钥$sk = r$。

$\operatorname{Enc}_{pk}(m)$：我们生成均匀随机(uniformly random)的$s \leftarrow \mathbb{Z}_q^n$，计算
$$(c_1, c_2) = (sA+e, su+e'+\frac{q}{2}m)$$
则$(c_1, c_2)$为密文。

$\operatorname{Dec}_{sk}(c_1, c_2)$：计算$\lfloor c_2 - c_1 \cdot r\rceil$。

我们考察解密正确性，即计算$c_2 - c_1 \cdot r$。

$$c_2 - c_1r = (su+e'+\frac{q}{2}m) - (sA+e)r$$
$$= sAr+se +e'+\frac{q}{2}m- sAr - er = \frac{q}{2}m + e' + se - er$$

其中后面的$e' + se - er$是小的。于是，我们只需要考虑$\frac{q}{2}m$是接近于0还是接近于$\frac{q}{2}$（在$\mod q$）意义下。

在这里。我们需要知道$r$是一个短向量。如果$r$是大的，那么无法保证解密的正确性。

------

下面，我们考虑上述PKE方案的安全性。注意到，公钥$pk = (A, u = Ar + e)$。于是根据LWE问题的安全性，计算$r$是困难的。

我们考察密文$\operatorname{Enc}_{pk}(m)$：$(c_1, c_2)=(sA+e, su + e' + \frac{q}{2} m)$。

于是根据DLWE问题的困难性，区分$(A, c_1 = sA+e)$和$(A, random)$是困难的。即，根据攻击者的能力是无法分辨出$c_1$和随机的区别的。

下面看$c_2$。在$c_2$中，根据LWE问题的困难性假设，我们可以证明$u$的不可区分性。但是，$su+e$是不可区分的。

------

这里介绍一个引理：Leftover Hash Lemma

**引理.** $A \leftarrow \mathbb{Z}_q^{n \times m}$，$r \leftarrow \{0, 1\}^m$，$u \leftarrow \mathbb{Z}_q^n$。如果$m > n\log q + O(\frac{1}{\log q})$，那么
$$(A, Ar)\approx (A, uniRandom)$$

即$(A, Ar)$和$(A, uniRandom)$是统计上不可区分的。

------

我们在这里不会去证明LHL。但是，我们提供一种理解LHL的思路：LHL本质上是一种随机性的提取或者降维(Randomness extraction)。

我们假设有一个随机源，它产生的东西不是真随机，但是总是带有一些伪随机的成分在里边。比如一个伪随机数生成器PRNG。我们知道，PRNG是一个$\{0, 1\}^m \rightarrow \{0, 1\}^n$的映射$f$，其中$n > m$。

$f$并不是真随机的（因为在$\{0, 1\}^n$上面，$f$最多能映射到$2^m$个值）。

假设存在一个不是真随机的随机源$r$。那么，$Ar$就是一个提取随机性的过程。$A$就是提取器。

假设$r$的熵是$m$。如果$u$是真随机的，那么$u$的熵就是$n\log q$。

因此，如果$m$比$n\log q$大一点点，那么$A$就是一个保留足够多随机信息的映射。我们可以在计算上认为它产生的东西就是随机的。

------

**定理.** 上述PKE是CPA安全的。

我们需要证明，$(pk, \operatorname{Enc}_{pk}(m))$和$(pk, Random)$是统计不可区分的。

$pk = (A, u = Ar)$，$\operatorname{Enc}_{pk}(m) = (sA+e, su+e'+\frac{q}{2}m)$

于是$pk = (A, u = Ar)$和$(A, u_0)$不可区分(LHL)。这里$u_0$是真·均匀随机。

于是$(sA+e, su+e'+\frac{q}{2}m)$和$(u_1, u_2+\frac{q}{2}m)$不可区分（因为LWE问题）。这里$u_1, u_2$是真 · 均匀随机。

最后，$u_2+\frac{q}{2}m$和$u_2$也是不可区分的。

------

PKE的另一种形式：

$\operatorname{KeyGen}$：
* $pk = (A, u)$，其中$A \leftarrow \mathbb Z_q^{m \times n} (m > n), u = As + e$。$A$是一个高矩阵。
* $sk = s$

$\operatorname{Enc}$：生成向量$r \leftarrow \{0, 1\}^m$。则$\operatorname{Enc}_{pk}(m) = (rA, ru + \frac{q}{2}m)$。注意到，这里我们没有加噪，则密文形式不再是LWE。

$\operatorname{Dec}$：给定密文$(c_1, c_2)$，计算$\lfloor c_2 - c_1s\rceil$。


（希望大家能够分清$m$作为维度和作为明文的区别。这并不严谨，但是暂时还不影阅响读）

解密的正确性：$c_2 - c_1s = ru + \frac{q}{2}m - rAs = rAs + re + \frac{q}{2}m - rAs = \frac{q}{2}m + re$

注意到$r$是小的。

安全性分析：我们将简述$(pk, Enc(m)) \approx (pk, u_0)$的证明思路，其中$u_0$是真·均匀随机。

$(A, u, c_1, c_2) = (A, u = As + e, c_1 = rA, c_2 = ru + \frac{q}{2}m)$

于是根据DLWE问题，无法区分

$$(A, As + e, rA, ru + \frac{q}{2}m)$$
和
$$(A, z_1, rA, rz_1 + \frac{q}{2}m)$$

根据LHL，无法区分$rA, rz_1$和均匀随机向量$z_2, z_3$的差别。

------

Lattice Trapdoor（格中的陷门）。陷门这个东西，可以拿来构造一些诸如签名、IBE、FHE的方案。

首先，什么是陷门？陷门可以认为是额外的信息（另一种未经证实的类比，陷门类似于软件的后门）。

给定LWE/SIS问题，如果我们有了陷门，那么求解SIS/LWE问题就是简单的。如果没有陷门，那么求解SIS/LWE问题就是难的。

我们以SIS问题为例。固定$A$, 任意给定$u$，我们想采样短向量$r$，使得$Ar = u$。如果$A$是随机矩阵，那么这就是SIS。它是困难的。

但是，如果我们换个顺序，给定$A, r$，计算$Ar = u$，这就是很简单的事情。

陷门的存在就支持我们做这样的更换顺序的事情。

GPV方案构造了一个陷门，但是比较复杂。MP12方案构建了一个非常漂亮的陷门：G-Trapdoor。其中，$G$是一个特定的矩阵。我们接下来将介绍MP12方案的相关细节。

我们以SIS问题为例。固定$G$, 任意给定$u$，我们想采样短向量$r$，使得$Gr = u$。如果$G$是随机矩阵，那么这就是SIS。但是如果$G$是特定的，那么这个事情**可能**很简单。

假设有这样的$G$，可以使得这个事情很简单。我们可以将一个$A$转化为$G$，即$\exists T$，使得$AT = G$。于是假设变成存在这样的$T$。

那么如何采样$z$，使得$Az = u$？

我们可以走如下的步骤：
1. 采样$r$，使得$Gr = u$
2. 输出$z = Tr$。于是$A(Tr) = u$。

此处，$T$为陷门。其实，给定随机的矩阵$A$，寻找$T$是很困难的。
------

LWE inversion：我们将基于LWE问题叙述其设计方案。

给定$G$和$sG+e$，我们想获得$s$和$e$。我们假设在这个$G$下，LWE是简单的：存在高效的方法基于$G$和$sG+e$找出$s$和$e$。

那么我们可否基于它构造一个LWE的陷门？假设我们知道$AT=G$，那么我们可以构造$sAT+eT = sG+e'$。

我们对于$T$的大小有一些要求：如果$T$过大，那么

假设我们用这个特殊的$G$，那么可以构造陷门对应的LWE问题是简单的，即存在问题是如何进行构造对应的陷门？


这一块看傻了。直接看构造方法：

1. 不带Trapdoor的LWE生成方法：$A \leftarrow \mathbb Z_q^{n \times m}$（A是一个矮胖的矩阵）
2. 构造Trapdoor：采样$(A,T)$。我们把$T$遮住，希望$A$看着像uniformly Random。MP12中的构造方法如下：
    
    构造$[A_1 A_2]$，其中$A_i \in Z_q^{n \times m}, i=1,2$。$A_1$来自均匀采样，$A_2 = A_1R+G$，其中$R \leftarrow \{0, 1\}^{m \times m}$。

几个问题：
1. 这里的$A$为什么和Uniformly random不可区分？因为LHL。
2. 怎么把$A$转换到$G$？
   
    我们构造$\begin{bmatrix}A_1 \ A_1R + G\end{bmatrix} \begin{bmatrix}-R \\ I\end{bmatrix} = G$，于是$T = \begin{bmatrix}-R \\ I\end{bmatrix}$。这里，矩阵$R$决定了$T$的范数。
3. 需要注意，$G$并不唯一。但是，我们可以给出一些构造$G$的方法。
4. $G$是什么，需要满足什么性质？给定$G$，$\forall u$，可以找到短向量$Gr = u$。当用$G$作为LWE中的矩阵时，LWE问题应当是简单的。

我们选取向量$g = [1\ 2\ 4\ 8\ \cdots \frac{q}{2}]$。于是$G = g \otimes I_n$。

有了这个向量$g$，我们可以计算$g \cdot r = a$。于是这个事情就变成了二进制分解。

同理，$Gr = u$也可以进行计算。

上面的步骤应该是在构造基于SIS的陷门。基于LWE的陷门的原理其实没有这么Trivial。

我们给定$sg + e$。现在的目标是，寻找$s$。

我们首先假设$q = 2^k$。



