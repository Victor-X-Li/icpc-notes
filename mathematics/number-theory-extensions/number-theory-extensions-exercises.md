# 数论延伸：习题分析

对应讲解：[广义 CRT 与一次式取整求和](number-theory-extensions.md)。建议先独立尝试，再按需阅读。

## 1. ABC193 E — Oversleeping

[原题](https://atcoder.jp/contests/abc193/tasks/abc193_e)

两个活动分别在每个周期 $2(X+Y)$ 的区间 $[X,X+Y)$、每个周期 $P+Q$ 的区间 $[P,P+Q)$ 内开启。求最早同时开启的时刻。

枚举第一周期内的活跃余数 $a=X,\ldots,X+Y-1$ 与 $b=P,\ldots,P+Q-1$。每对解广义 CRT：

$$t\equiv a\pmod{2(X+Y)},\qquad t\equiv b\pmod{P+Q}.$$

设两周期为 $M_1,M_2$，代入 $t=a+M_1k$，得到 $M_1k\equiv b-a\pmod{M_2}$；令 $d=\gcd(M_1,M_2)$，仅当 $d\mid b-a$ 时有解，约分求逆得到最小非负 $k$，再把 $t$ 归一化到 $[0,\operatorname{lcm}(M_1,M_2))$。所有可行余数对的最小解取最小值，全部无解输出 `infinity`。

枚举为什么完整？任意同时开启的整数时刻在两个周期内各有一个活跃余数；每个 CRT 解又确实落在对应活跃时段。半开区间右端不活跃，因此不能枚举到 $X+Y$ 或 $P+Q$。

题目允许枚举的是短活跃区间 $Y,Q\le500$，不是长度可能很大的整个周期。时间 $O(YQ\log V)$、额外空间 $O(\log V)$（递归扩欧）。周期乘积及 CRT 中间乘法用足够宽的整数，先除 GCD 再乘可减少溢出。

## 2. AtCoder Library Practice Contest C — Floor Sum

[原题](https://atcoder.jp/contests/practice2/tasks/practice2_c) · [官方接口与实现](https://atcoder.github.io/ac-library/production/document_en/math.html)

每组给出 $N,M,A,B$，求

$$
F(N,M,A,B)=\sum_{i=0}^{N-1}\left\lfloor\frac{Ai+B}{M}\right\rfloor.
$$

原题 $N,M\le10^9$，$0\le A,B<M$，最多 $10^5$ 组。逐项累加不可行；这里只验收自行实现，不把调用 `atcoder::floor_sum` 当作推导过程。

先按正文第 2 节拆出 $A,B$ 中整倍数部分：

$$
F(N,M,A,B)=\left\lfloor\frac AM\right\rfloor\frac{N(N-1)}2+
\left\lfloor\frac BM\right\rfloor N+F(N,M,A\bmod M,B\bmod M).
$$

若 $N=0$，立即返回零；归约后若 $A=0$，剩余和也为零。否则设

$$
H=\left\lfloor\frac{A(N-1)+B}{M}\right\rfloor.
$$

按高度数整数点，得到

$$
F(N,M,A,B)=NH-F(H,A,M,M-B+A-1).
$$

完整换序与阈值证明见[正文第 2 节](number-theory-extensions.md#2-一次式取整求和怎样再次回到欧几里得算法)。每轮的整倍数部分直接计算；剩余部分用这个等式继续。归约后的 $A<M$，交换后模数变为 $A$，随后再取模，因此按欧几里得算法下降。递归中的 $H=0$ 也允许，不能在下一层计算 $N-1$ 后才处理空和。

每组时间 $O(\log M)$，递归空间 $O(\log M)$。原题答案小于 $N^2\le10^{18}$；原题范围内上述递推可用有符号 64 位保存。实现若推广到更大范围，须重新核对 $A(N-1)+B$、$NH$ 以及整数部分的乘法，不能只检查最终返回值。计算 $N(N-1)/2$ 时可先对偶数因子除二。

可手查 $N=4,M=10,A=6,B=3$：四项为 $0,0,1,2$，总和为 3。$M=1$ 时原题强制 $A=B=0$，答案为零；$N=1$ 时答案就是 $\lfloor B/M\rfloor=0$。这些边界可以检查空和与零斜率分支。
