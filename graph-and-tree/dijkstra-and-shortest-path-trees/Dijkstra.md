Dijkstra claim that，每次从未确认的结点集合 U 中选择 $d(u)$ 最小的结点 $u$ 时，便有 $d(u) = \delta(s, u)$。

初始，$U = V$，$d(s) = 0$，其他结点 $d(i) = \infty$



用归纳法证明 claim，一开始，选择结点 $s$，trivial

如果选择了结点 $v$，由于 $d(v)$ 是由一条 $s \to v$ 的真实存在的路径更新的，一定有 $d(v) \geq \delta(s, v)$

我们选择，$s \to v$ 的**任意**一条最短路 $p(s, v)$，由于 $s$ 已选择，$v$ 未选择，$p(s, v)$ 存在至少一个结点未确定，取第一个未确定的结点，设为 $x$（$x$ 可以为 $v$），$x$ 的前驱设为 $y$

- 由于 $w \geq 0$，必定有 $\delta(s, x) \leq \delta(s, v)$
- 由于 $x$ 被 $y$ 更新，有 $d(x) = \delta(s, x)$
- 又由于我们选择的是 $v$ 而不是 $x$，有 $d(v) \leq d(x)$

综合成不等式 $d(v) \leq d(x) = \delta(s, x) \leq \delta(s, v)$

结合前面，有 $d(v) = \delta(s, v)$，claim 成立



尝试思考，图需要满足什么性质，才能使用 Dijkstra 算法？

- 如果存在 $(u, v)$ 的有向边，必须有 $\delta(s, u) \leq \delta(s, v)$。对于 $w < 0$ 的边，这条性质不满足
- **任意**最短路的**任意**前缀都是最短路？这个条件过于强了，Dijkstra 也可以处理前缀 $\max$ 问题，但前缀 $\max$ 问题不满足这个性质。应该改写成：？？？
