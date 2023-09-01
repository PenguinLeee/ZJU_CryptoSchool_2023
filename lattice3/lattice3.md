# Lattice 3-4: LWE for PKE\IBE\FHE, ideal lattice and RLWE

在这一篇文章中，我们将涉及：
1. $LWE \leq PKE$，此即，我们可以基于LWE构造公钥加密
2. 进一步地，我们如何构造签名？IBE？FHE？怎么优化参数？
3. 如何基于理想格/RLWE做一些事情？

此即：构造
$$LWE \leq PKE$$
$$LWE \leq SIG$$
$$LWE \leq FHE$$
（在后两者中，我们将使用格Trapdoor技巧）

------

首先，我们回顾Oded Regev的PKE构造方案（的一种构造方式）。它是很多IBE/FHE方案的基础。

$\operatorname{KeyGen}$：我们生成$A \in \mathbb Z_q^{n \times m}$。（这里需要$A$是一个矮胖的矩阵，即$n < m$）。$r$是一个短向量，不妨认为$r \in \{0, 1\}^m$。计算$u = Ar$。
于是：
* 公钥$pk = (A, u = Ar)$
* 私钥$sk = r$。

$\operatorname{Enc}_{pk}(m)$：我们生成均匀随机(uniformly random)的$s \leftarrow \mathbb{Z}_q^n$，计算
$$(c_1, c_2) = (sA+e, su+e'+\frac{q}{2}m)$$
其中$e, e'$都是噪声。则$(c_1, c_2)$为密文。

$\operatorname{Dec}_{sk}(c_1, c_2)$：计算$\lfloor c_2 - c_1 \cdot r\rceil$。其中$\lfloor ·\rceil$是取整。

我们考察解密正确性，即计算$c_2 - c_1 \cdot r$。

$$c_2 - c_1r = (su+e'+\frac{q}{2}m) - (sA+e)r$$
$$= sAr+e'+\frac{q}{2}m- sAr - er = \frac{q}{2}m + e' - er$$

其中后面的$e' + se - er$是小的。于是，我们只需要考察$\frac{q}{2}m$是接近于0还是接近于$\frac{q}{2}$（在$\mod q$意义下）。

在这里。我们需要知道$r$是一个短向量。如果$r$太大，那么无法保证解密的正确性。

------

下面，我们考虑上述PKE方案的安全性。我们首先观察密文的形式：

$$(c_1, c_2) = (sA+e, su+e'+\frac{q}{2}m)$$

注意到$c_1, c_2$长得都比较像LWE的sample。$c_1$显然就是一个LWE。如果$u$是uniformly random的向量，那么$c_2$也将是一个LWE的sample。

但是$u=Ar+e$，从构造上来看，$u$其实并不是在$\mathbb Z_q^n$上uniformly random的。因为$r$是在$\{0, 1\}^m$上均匀随机的而不是在$\mathbb Z_q^n$上均匀随机的。

但是$u实际上和uniformly random（在统计上）不可区分的。

为了说明这样一件事情，我们引入Leftover Hash Lemma. 

它大概是说，你如果从$\{0, 1\}$中均匀随机选$r$，那么$u$的分布和均匀随机（在统计上）不可区分。

------

**引理（Leftover Hash Lemma, LHL）.** $A \leftarrow \mathbb{Z}_q^{n \times m}$，$r \leftarrow \{0, 1\}^m$，$u \leftarrow \mathbb{Z}_q^n$。如果$m > n\log q + O(\frac{1}{\log q})$，那么
$$(A, Ar)\approx_\epsilon (A, u)$$

其中，$\approx_\epsilon$指统计距离的差为$\epsilon$。

即，$(A, Ar)$和$(A, u)$是统计上不可区分的。

------

我们在这里不会去证明LHL。但是，我们提供一种理解LHL的思路：LHL本质上是一种对随机性的提取或者降维(Randomness extraction)。

假设存在一个随机源$r$。这个$r$均匀采样于$\{0, 1\}^m$，但是如果把这个$r$放在$\mathbb Z_q^m$下，那么它显然不是$\mathbb Z_q^m$上均匀随机的。

我们计算$Ar$。这个$Ar$就是一个提取（或者压缩）随机性的过程。矮胖的矩阵$A$就是提取器。

$r$的熵是$m\log 2$。如果$u$是真随机的，那么$u$的熵就是$n\log q$。（取对数底数为2）

因此，如果$m\log 2$比$n\log q$大一点点，那么可以认为$r$中包含的随机性足够萃取成一个$\mathbb Z^n_q$上的随机向量。我们就可以在计算上认为$Ar$就是随机的。

从这里的分析来看，引理中涉及的不等式$m > n\log q + O(\frac{1}{\log q})$似乎是存在一些不准确的地方。


------

**定理.** 上述PKE是安全的，并且密文是伪随机的。

我们需要证明，$(pk, \operatorname{Enc}_{pk}(m))$和$(pk, uniform)$是统计不可区分的。

$(pk, \operatorname{Enc}_{pk}(m)) = (A, u = Ar, sA+e, su+e'+\frac{q}{2}m)$，

我们有$pk = (A, u = Ar)$和$(A, u_0)$不可区分(LHL)。这里$u_0$是真·均匀随机。

于是$(pk, \operatorname{Enc}_{pk}(m))$和$(A, u_0, sA+e, su+e'+\frac{q}{2}m)$不可区分。

根据LWE问题（和LHL），$(A, u_0, sA+e, su+e'+\frac{q}{2}m)$和$(A, u_0, u_1, u_2+\frac{q}{2}m)$是不可区分的。其中$u_1, u_2$是真·均匀随机。

最后，由于一次性密码本，$(A, u_0, u_1, u_2+\frac{q}{2}m)$和$(A, u_0, u_1, u_2)$是不可区分的。

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

To be continue...


------

这里，我们构造一个基于SIS问题的签名：

我们考虑$\operatorname{Sign}(m)$。矩阵$A$，$A \cdot r_m = H(m)$。其中，$H(m)$是$m$的哈希值。

关于验签，我们只需要验证两件事：
1. $r_m$是一个短向量；
2. $A \cdot r_m = H(m)$。

**定理.** 在SIS问题的安全性假设以及random oracle的安全性假设下，签名函数$\operatorname{Sign}(·)$是安全的。

**证明思路.** 我们假设存在攻击者$\mathcal A$，他可以攻破这个签名函数。

我们的证明目标是，攻击者可以攻破SIS。


To be continue...

------

FHE（Fully Homomorphic Encryption）

首先，看一下全同态加密的概念。

我们任意选取若干的明文$x_1,...,x_n$。与此同时有一个特殊的加密方案$(KeyGen, Enc, Dec)$。我们可以计算一个函数$f(x_1,...,x_n)$。现在我们分别对这些明文进行加密，得到$c_i = Enc(x_i), i=1,...,n$。这个特殊加密方案还支持对密文$c_1,...,c_n$做一些密文的运算$EncEval(f)$。计算$EncEval(f)(c_1, ..., c_n)$之后解密的结果，和在明文状态下计算$f(x_1,...,x_n)$的结果是一样的。这就是同态加密的概念。

同态加密到目前为止经历了四代：
* 第一代：Gentry在2009年提出的基于ideal lattice的加密方案。
* 第二代：BV, BGV, BFV等方案——基于RLWE问题，支持对一个整数向量进行加密，进而支持SIMD运算。
* 第三代：GSW，一个非常简洁的方案。FHEW, TFHE等。
* 第四代：CKKS方案，支持浮点向量的运算。但是，第四代和第二代的方法没有什么本质上的差别，主要是编码消息时使用的技巧(基于Canonical Embedding)有所不同。但是这个方案因为可以对浮点数进行操作，所以在隐私保护机器学习中还是比较火的。

这一部分将基于GSW方案进行介绍。

这个方案有一个特征，就是在$KeyGen$时需要对外暴露两个Key。一个是公钥$pk$，一个是在做同态计算时用到的密钥$evk$。

下面，我们来看具体的实现。

$\operatorname{KeyGen}$. 首先生成随机矩阵$A$，采样私钥$sk = s$，计算公钥$pk = \begin{bmatrix}A \\
 b\end{bmatrix} = B$，其中$b = sA+e$，$e$是误差。可以根据LHL证明$pk$的伪随机性。

$\operatorname{Enc}$. 对比特$m$进行加密。给定比特$m \in \{0, 1\}$，计算$C= BR + mG$，其中$R \in \{0, 1\}^{M \times M}$。

$\operatorname{Dec}$. 假如我们直接计算$(-s, 1) · C = (-s, 1)BR + (-s, 1)mG = eR + (-s, 1)mG$，我们会发现仍然无法计算出$m$。其中，$(-s, 1) · B = b - sA = e$。

下面的问题在于如何提取$m$。我们找一个向量$r$，使得
$$Gr = \begin{bmatrix}
0 \\
\vdots \\
0 \\
q/2 
\end{bmatrix}$$

于是有：$(-s, 1) · Cr = (-s, 1)BRr + (-s, 1)mGr = e'' + mq/2$。只需要考察$e'' + mq/2$更接近0还是$q/2$就可以了。

------

下面，我们考察加法的计算。假设有$C_1 = BR_1 + m_1G$和$C_2 = BR_2 + m_2G$。

可以直接得到：$C_1 + C_2 = B(R_1+R_2) + (m_1+m_2)G$。

但是做乘法不是那么容易。假设有$C_1 = BR_1 + m_1G$和$C_2 = BR_2 + m_2G$。我们所希望的是，做乘法之后的密文仍然有类似于$BR + mG$的密文形式。

最后达到的效果类似于计算$C_1G^{-1}(C_2)$。

首先，我们需要找到一个计算$G^{-1}$的方法。基于比特分解我们可以得到$GG^{-1} = I$。

于是计算密文乘法的过程就类似于：

$$(BR_1 + m_1G)G^{-1}C_2$$
$$= BR_1G^{-1}C_2 + m_1GG^{-1}C_2$$
$$= BR'+m_1BR_2+m_1m_2G $$
$$= BR' + Bm_1R_2 + m_1m_2G $$
$$= BR'' + m_1m_2G$$

于是转化成了我们想要的形式。

我们考察上述的乘法操作。假如给定密文$C_1, C_2$，它们的噪声界分别为$b_1, b_2$，那么你计算$C_1C_2$和计算$C_2C_1$的结果存在差别，因为乘法的先后次序不同，引入的噪声是不一样的。

$G^{-1}$大概将噪声扩大了$\sqrt{n}$倍。
