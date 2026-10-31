# 数论延伸：广义 CRT 与一次式取整求和

这篇从[数论主篇](../number-theory/number-theory.md)移出两部分：非互质模数的 CRT，以及原第 10 节的 floor_sum。两部分互不依赖，可以分别阅读。

广义 CRT 以前面的线性同余和普通 CRT 为基础；取整求和只需要整数除法、按对象交换求和与欧几里得算法。普通 CRT 的构造、证明及练习继续保留在主篇第 6 节。

## 1. 广义 CRT：把两个剩余类合并

设已经有两条约束

$$
x\equiv a\pmod m,\qquad x\equiv b\pmod n.
$$

第一条告诉我们 $x=a+mt$。代入第二条，就得到熟悉的线性同余

$$
mt\equiv b-a\pmod n.
$$

令 $g=\gcd(m,n)$，于是

$$
\boxed{\text{两式兼容}\iff g\mid(b-a).}
$$

兼容时，约分后求出

$$
t\equiv\frac{b-a}{g}\left(\frac mg\right)^{-1}\pmod{n/g}.
$$

取一个代表 $t_0$，就得到

$$
\boxed{x\equiv a+mt_0\pmod{\operatorname{lcm}(m,n)}.}
$$

这既是构造，也是全部解的证明：$t$ 每增加 $n/g$，$x$ 增加 $mn/g$；而线性同余告诉我们没有其他 $t$。因此不只找到了某个交点，还把整组交点重新压成了一条等差数列。

例如 $x\equiv2\pmod6$、$x\equiv5\pmod9$。设 $x=2+6t$，得到 $2t\equiv1\pmod3$，所以 $t\equiv2\pmod3$，最终

$$
x\equiv14\pmod{18}.
$$

若第二式改为 $x\equiv4\pmod9$，两条约束在模 $3$ 下要求不同结果，就无解。非互质模数下的问题不是“公式要多补一个逆元”，而是约束本身可能冲突。

![非互质模数的余数组合不一定兼容](../../asset/nt-crt-compatible-residues.png)

图中横纵坐标分别是模 $6$、模 $9$ 的余数。一个整数只会落在两坐标模 $3$ 相同的格子里。蓝点共有 $18=\operatorname{lcm}(6,9)$ 个，与模 $18$ 的全部整数一一对应；并不是 $6\times9$ 个格子都可取。

### 为什么互质模数能够真正独立选择？

当 $\gcd(m,n)=1$，兼容条件自动满足。因此每一对余数 $(a\bmod m,b\bmod n)$ 都唯一对应一个模 $mn$ 的整数。这才是“先在两个模数下独立求解，再组合答案”的依据。

特别地，$M=\prod p^{e_p}$ 的不同质数幂两两互质。若某个问题的条件能逐个模 $p^{e_p}$ 检查，可以分别处理，再用 CRT 合并。这与[数论主篇第 4 节](../number-theory/number-theory.md#4-唯一分解乘法约束可以改成每个质因子上的大小关系)的指数分解相呼应，但并不是所有关于整数的条件都能这样拆：普通大小关系、整数加和范围不属于单独一个剩余类的信息。

多条同余可以顺次合并，只维护当前的最小非负代表和模数。一次合并使用扩展欧几里得，耗时为模数大小的对数级；$k$ 条约束在最终 LCM 适合固定宽度整数的前提下，可界为 $O(k\log M)$。有约分模数 $n/g=1$ 时，该条已经被包含，取 $t_0=0$ 即可。

得到 $x\equiv r\pmod M$ 后，如果题目还要求 $x\ge L$，最小解是

$$
r+M\left\lceil\frac{L-r}{M}\right\rceil.
$$

区间 $[L,R]$ 内的解数为

$$
\left\lfloor\frac{R-r}{M}\right\rfloor-
\left\lfloor\frac{L-1-r}{M}\right\rfloor.
$$

CRT 负责把等式约束合并，区间与最小值要求再沿这条等差数列处理。这比为了“最早同步”直接模拟到某个巨大时间可靠得多。

计算 $\operatorname{lcm}(m,n)$ 应先除以 GCD 再乘；即使最终 LCM 能保存，构造解时的中间乘积也可能更大。公式正确与所用整数类型能否承载运算，是两件要分别核查的事。

> **练习**：[ABC193 E — Oversleeping](https://atcoder.jp/contests/abc193/tasks/abc193_e)（估算约 2300；2021）。关键在于识别哪些周期内的候选足够少，可以先枚举，再合并各自的同余条件。完整分析见[配套第 1 题](number-theory-extensions-exercises.md#1-abc193-e--oversleeping)。

## 2. 一次式取整求和：怎样再次回到欧几里得算法？

[数论主篇第 9 节](../number-theory/number-theory.md#9-整除分块商相同的区间可以一次处理)的整除分块处理 $\lfloor N/i\rfloor$；另一种常见结构则是

$$
F(n,m,a,b)=\sum_{i=0}^{n-1}\left\lfloor\frac{ai+b}{m}\right\rfloor,
$$

其中 $n,a,b\ge0,m\ge1$。不要因为它也含有取整，就照搬整除分块的分块式：现在变化的是分子，结构不一样。

先把明显的整数部分拆出来。写 $a=qm+a'$、$b=rm+b'$，则

$$
F(n,m,a,b)=q\frac{n(n-1)}2+rn+F(n,m,a',b').
$$

于是只需处理 $0\le a,b<m$。若 $n=0$ 或 $a=0$，这时剩余和为零。否则令

$$
h=\left\lfloor\frac{a(n-1)+b}{m}\right\rfloor.
$$

把每一项的商想成该列下方的整数点数。固定高度 $y\in[1,h]$，这层哪些列被计算？需要

$$
my\le ai+b
\iff i\ge\left\lceil\frac{my-b}{a}\right\rceil.
$$

因为 $b<m$，这个阈值为正；又因为 $y\le h$，它不超过 $n-1$。所以本层有 $n-\lceil(my-b)/a\rceil$ 列。改为逐层求和，并令 $j=y-1$，得到

$$
\boxed{F(n,m,a,b)
=nh-F(h,a,m,m-b+a-1).}
$$

若 $h=0$，右侧也直接为零。关键在于右边的**新模数是原来的 $a$，严格小于 $m$**。下一次再拆出分子的整数部分，就出现 $m\bmod a$；参数像欧几里得算法一样下降，总递归层数为 $O(\log m)$。这一式与常用 floor_sum 模板的写法可能不同，但数的是同一批整数点。

例如 $n=4,m=5,a=2,b=1$，原和为 $0+0+1+1=2$。$h=1$，上式给出

$$
4-F(1,2,5,5)=4-2=2.
$$

这段推导值得理解的地方不是背递推式，而是：**按列不好求和，就改成按高度数；交换以后，原来的斜率分子与分母互换。** 本篇只讨论非负参数，负参数必须先用数学意义的下取整归约，不能直接依赖向零截断的整数除法。

如果需要检查接口或实际实现，可以参考 [AtCoder Library 的 floor_sum](https://atcoder.github.io/ac-library/production/document_en/math.html)。使用前核对它的参数范围和返回值溢出约定；数学公式本身不保证所有中间结果都适合 64 位。

> **实现验收**：[AtCoder Library Practice Contest C — Floor Sum](https://atcoder.jp/contests/practice2/tasks/practice2_c)（估算约 2200，按自行推导并实现算法评价；直接调用现成库不代表同等训练难度）。本题直接检验这一节的递推，完整分析见[配套第 2 题](number-theory-extensions-exercises.md#2-atcoder-library-practice-contest-c--floor-sum)。VJudge 对应页暂未核对，保留原题入口。

## 参考

- [AtCoder Library：Math](https://atcoder.github.io/ac-library/production/document_en/math.html)：广义 CRT 与 floor_sum 的接口、范围及复杂度。本文的按高度计数递推是原 notes 的独立推导，不要求与库实现逐行相同。
- [ABC193 E 官方解说](https://atcoder.jp/contests/abc193/editorial/818)：周期中的短活跃区间枚举及同余合并。
- [AtCoder Library Practice Contest C](https://atcoder.jp/contests/practice2/tasks/practice2_c)：floor_sum 的直接实现验收。
