# ICPC 数论：习题分析

对应讲解：[number-theory.md](number-theory.md)。本文覆盖原文全部练习；已在正文展开的具名例题也保留分析入口。建议先独立尝试，再按需阅读。默认不附完整代码。

## 1. CF1458 A — Row GCD

[原题](https://codeforces.com/problemset/problem/1458/A)

每次将所有 $a_i$ 加同一个 $b$，求它们的 GCD。利用 $\gcd(x,y)=\gcd(x,y-x)$，将共同变化消去：

$$\gcd(a_1+b,\ldots,a_n+b)=\gcd(a_1+b,a_2-a_1,\ldots,a_n-a_1).$$

预处理 $g=\gcd_{i\ge2}|a_i-a_1|$，每个询问输出 $\gcd(a_1+b,g)$。相同变化没有改变任意两项差，因此这些差正是可复用的信息。

若 $n=1$ 或所有 $a_i$ 相等，$g=0$，$\gcd(x,0)=|x|$ 自动给出答案。取绝对值避免依赖某个实现对负 GCD 的约定。设数值规模为 $V$，时间 $O((n+m)\log V)$；除存询问外可只用 $O(1)$ 工作空间。加法与差值都要能装进整数类型。

## 2. ABC186 E — Throne

[原题](https://atcoder.jp/contests/abc186/tasks/abc186_e)

从余数 $S$ 出发，每次加 $K$，模 $N$，求首次到 0 的非负步数 $x$。方程为

$$Kx\equiv-S\pmod N.$$

令 $d=\gcd(K,N)$。若 $d\nmid S$，不可达；否则除以 $d$，得到

$$\frac Kd x\equiv-\frac Sd\pmod{N/d}.$$

约分后两系数互质，可用扩展欧几里得求逆元。所有步数构成模 $N/d$ 的一个剩余类，取其最小非负代表就是最少步数。注意负数取模的归一化，应得到 $[0,N/d)$ 的代表。

若 $N/d=1$，约分后的同余不再限制 $x$，答案直接为 0，不必向逆元接口传模数 1。每个测试 $O(\log N)$ 时间、迭代扩欧可用 $O(1)$ 工作空间。直接模拟会走到 $N/d$ 步，数值大时不可行。

## 3. ABC280 D — Factorial and Multiple

[原题](https://atcoder.jp/contests/abc280/tasks/abc280_d)

求最小 $n$ 使 $K\mid n!$。分解 $K=\prod p^{e_p}$ 后，判定等价于每个质因子的指数都够：

$$v_p(n!)=\sum_{t\ge1}\left\lfloor\frac n{p^t}\right\rfloor\ge e_p.$$

前一层统计每个 $p$ 的倍数提供的第一个 $p$，后一层补上 $p^2$ 倍数多出的一个，依此类推，所以不会漏掉高次幂。计算时反复令 $q\gets\lfloor q/p\rfloor$ 并累加，而不是生成可能溢出的 $p^t$。

判定随 $n$ 单调，二分 $[1,K]$：因为 $K$ 自己是 $K!$ 的因子，右端一定可行。也可逐个质因子求所需最小 $n_p$，最终取最大值，逻辑相同。

原题 $K\le10^{12}$，试除分解 $O(\sqrt K)$ 足够，注意试除后剩余的大质数。设不同质因子个数为 $r$，二分判定总时间上界 $O(r\log^2K)$，总计 $O(\sqrt K+r\log^2K)$，空间 $O(r)$。不能把“$K$ 中最大质数”当答案，例如 $K=8$ 需要 $4!$ 而非 $2!$。

## 4. ABC193 E — Oversleeping

[原题](https://atcoder.jp/contests/abc193/tasks/abc193_e)

两个活动分别在每个周期 $2(X+Y)$ 的区间 $[X,X+Y)$、每个周期 $P+Q$ 的区间 $[P,P+Q)$ 内开启。求最早同时开启的时刻。

枚举第一周期内的活跃余数 $a=X,\ldots,X+Y-1$ 与 $b=P,\ldots,P+Q-1$。每对解广义 CRT：

$$t\equiv a\pmod{2(X+Y)},\qquad t\equiv b\pmod{P+Q}.$$

设两周期为 $M_1,M_2$，代入 $t=a+M_1k$，得到 $M_1k\equiv b-a\pmod{M_2}$；令 $d=\gcd(M_1,M_2)$，仅当 $d\mid b-a$ 时有解，约分求逆得到最小非负 $k$，再把 $t$ 归一化到 $[0,\operatorname{lcm}(M_1,M_2))$。所有可行余数对的最小解取最小值，全部无解输出 `infinity`。

枚举为什么完整？任意同时开启的整数时刻在两个周期内各有一个活跃余数；每个 CRT 解又确实落在对应活跃时段。半开区间右端不活跃，因此不能枚举到 $X+Y$ 或 $P+Q$。

题目允许枚举的是短活跃区间 $Y,Q\le500$，不是长度可能很大的整个周期。时间 $O(YQ\log V)$、额外空间 $O(\log V)$（递归扩欧）。周期乘积及 CRT 中间乘法用足够宽的整数，先除 GCD 再乘可减少溢出。

## 5. CF1295 D — Same GCDs

[原题](https://codeforces.com/problemset/problem/1295/D)

计 $0\le x<m$ 中满足 $\gcd(a+x,m)=\gcd(a,m)$ 的数量。设 $d=\gcd(a,m)$，目标要求 $a+x$ 为 $d$ 的倍数，除以 $d$ 后又与 $m/d$ 互质。

$a+x$ 遍历长度为 $m$ 的连续区间 $[a,a+m)$，其中恰有 $m/d$ 个 $d$ 的倍数；它们除以 $d$ 后是连续的 $m/d$ 个整数，恰好覆盖模 $m/d$ 的全部剩余类。因此答案为

$$\boxed{\varphi(m/d).}$$

不是 $\varphi(m)$，也不需要求区间起点的复杂互质计数，因为这里刚好是一整个周期。分解 $v=m/d$，对每个不同质因子执行 $ans\gets ans/p\cdot(p-1)$。

试除时间 $O(\sqrt v)$、工作空间 $O(1)$；$v=1$ 时答案为 $\varphi(1)=1$。这个边界对应整个区间里只有一个 $d$ 的倍数可达目标 GCD。

## 6. CF1750 D — Count GCD

[原题](https://codeforces.com/problemset/problem/1750/D)

要求构造 $1\le b_i\le m$，使每个前缀 GCD 恰为给定 $a_i$。首项强制 $b_1=a_1$；每步必须满足 $a_i\mid a_{i-1}$，否则无解。

当该整除链成立时，前缀已经达到 $a_{i-1}$，只需

$$\gcd(a_{i-1},b_i)=a_i.$$

设 $q=a_{i-1}/a_i$，写 $b_i=a_ix$，则条件变成

$$1\le x\le\lfloor m/a_i\rfloor,\qquad\gcd(x,q)=1.$$

把 $q$ 的不同质因子集合记为 $P$，容斥给出本步选择数

$$c_i=\sum_{S\subseteq P}(-1)^{|S|}\left\lfloor\frac{\lfloor m/a_i\rfloor}{\prod_{p\in S}p}\right\rfloor.$$

答案是 $\prod_{i=2}^nc_i$，对 $998244353$ 取模；首项若超出 $[1,m]$ 则为零。相邻约束只通过规定的前缀 GCD 传递，所以各步选择数可以相乘，不需要记录更早的 $b$。

所有比值都是 $a_1$ 的约数，可先分解 $a_1$ 一次，再从其质因子中筛出整除当前 $q$ 的那些。若 $q=1$，只有空集项，本步所有倍数都合法。设 $r=\omega(a_1)$，时间上界 $O(\sqrt{a_1}+nr+n2^r)$、空间 $O(r+2^r)$；原题 $a_i\le10^9$，不同质因子很少。枚举子集乘积时注意用宽整数。

## 7. ABC162 E — Sum of gcd of Tuples (Hard)

[原题](https://atcoder.jp/contests/abc162/tasks/abc162_e)

这是正文已展开的例题。长 $n$ 元组每项在 $[1,K]$，求 GCD 总和。令 $F(d)=\lfloor K/d\rfloor^n$，它数所有元素都被 $d$ 整除的元组；令 $E(d)$ 数 GCD 恰为 $d$ 的元组，则

$$E(d)=F(d)-\sum_{j\ge2,\ jd\le K}E(jd).$$

从大到小求完后答案为 $\sum dE(d)$。同一元组属于哪些 $F(d)$ 只取决于其 GCD 的约数，因此按倍数扣除恢复精确类别，不能按大小相邻相减。

若只要加权总和，还可用 $g=\sum_{d\mid g}\varphi(d)$ 得到

$$\boxed{\sum_{d=1}^K\varphi(d)\lfloor K/d\rfloor^n.}$$

对每个元组固定其 GCD 后展开这个恒等式，再交换求和即得。两式计算同一目标，但前者还保留完整 GCD 分布。

第一种时间 $O(K\log K+K\log n)$、空间 $O(K)$；第二种先筛欧拉函数，再求 $K$ 次快速幂，采用线性筛为 $O(K+K\log n)$、空间 $O(K)$。所有扣除都在模 $10^9+7$ 下归一化。$K=1$ 时唯一元组贡献为 1。

## 8. CF2037 G — Natlan Exploring

[原题](https://codeforces.com/problemset/problem/2037/G)

从较小下标走到较大下标，只有 $\gcd(a_i,a_j)>1$ 才有边，求从 1 到 $n$ 的路径数。显式检查所有边是平方的，但一条边存在等价于两个数共享至少一个质因子。

令 $dp[i]$ 为到 $i$ 的路径数，维护 $sum[d]$：已经处理的下标 $j$ 中，$d\mid a_j$ 的 $dp[j]$ 总和。将 $a_i$ 的不同质因子集合记为 $P$，对它们做加权容斥：

$$\boxed{dp[i]=\sum_{\varnothing\ne S\subseteq P}(-1)^{|S|+1}sum\!\left[\prod_{p\in S}p\right].}$$

固定一个以前的点 $j$，若它共享 $t>0$ 个质因子，最终系数为 $\sum_{s=1}^t(-1)^{s+1}\binom ts=1$；若不共享则系数为零。因此恰好把所有合法前驱的权重各取一次。

设 $dp[1]=1$，处理它时只做更新；每个点先查询再把自身 $dp[i]$ 加入全部非空质因子子集乘积，避免计入自环。$a_i=1$ 时没有前驱可连，除起点外 $dp[i]=0$，也不更新任何非空乘积。

原题值域 $V\le10^6$，筛最小质因子后分解。令 $r_i=\omega(a_i)$，时间 $O(V+\sum_i(\log a_i+2^{r_i}))$（线性筛），空间 $O(V)$，模 $998244353$。这里维护的是路径权重和，不能只记每个质数出现了几次。

## 9. CF616 E — Sum of Remainders

[原题](https://codeforces.com/problemset/problem/616/E)

求 $\sum_{i=1}^m(n\bmod i)$。先写

$$\boxed{nm-\sum_{i=1}^mi\lfloor n/i\rfloor.}$$

$i>n$ 时商为零，对减项无贡献，所以分块只需做到 $T=\min(n,m)$。当左端为 $l$，商 $q=\lfloor n/l\rfloor$ 在

$$[l,r],\qquad r=\min(T,\lfloor n/q\rfloor)$$

保持不变，该块减去 $q(l+r)(r-l+1)/2$，之后跳到 $r+1$。

块内权重是下标之和，不是块长；不能像无权取整和那样只乘 $r-l+1$。$i>n$ 的尾部余数都为 $n$，已经包含在最初的 $nm$ 中，不能先把 $m$ 截成 $n$ 再算初项。

不同正商只有 $O(\sqrt n)$ 个，所以时间 $O(\sqrt n+\log P)$、空间 $O(1)$。模 $P=10^9+7$ 时用逆元 2，或者在整数中先对偶因子除二再取模；不要先取余再普通除二。原题数值可达 $10^{13}$，$nm$ 在 64 位中会溢出，应先分别取模后乘，或用更宽中间类型。$m=1$ 时答案为零，可用于检查首块。
