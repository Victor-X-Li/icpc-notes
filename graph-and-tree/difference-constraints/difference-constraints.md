# SPFA 与差分约束的解

已知 Bellman–Ford、队列优化和负环检测。这里关心的是，给定一组约束，SPFA 返回的距离究竟把什么量推到了极值。

## 路径给出变量差的上界

把 $x_v-x_u\le w$ 写成 $x_v\le x_u+w$，连边 $u\xrightarrow{w}v$。松弛就是把过大的 $d_v$ 降到这条边允许的上界。下界 $x_v-x_u\ge L$ 可以改写为 $x_u-x_v\le -L$，于是连反向边 $v\xrightarrow{-L}u$。等式拆成两个方向的约束。整数变量的严格上界 $x_v-x_u<c$ 在 $c$ 为整数时可以写成 $x_v-x_u\le c-1$。

沿一条从 $u$ 到 $v$ 的路径把不等式相加，中间变量抵消，得到 $x_v-x_u\le$ 路径权和。每条路径都给出一个上界，最短路给出的上界最紧。沿负环相加则得到 $0\le$ 一个负数，因此全系统无解。

反过来，若没有负环，加超级源 $s$，向每个原点连零权边。这样每个点的最短距离都有限，且每条原边满足 $d_v\le d_u+w$，所以 $x_v=d_v$ 是一组解。权值为整数时，这组解也为整数。实现时可省去超级源，把所有原点的距离初始化为 $0$ 并全部入队，等价于先松弛超级源的所有出边。

练习 [P5960 差分约束](https://www.luogu.com.cn/problem/P5960)（估 CF 1400）。用它检查输入不等式的方向，以及互不连通的点是否都参与判环。[分析](difference-constraints-exercises.md#p5960)

进阶练习 [CF241E Flights](https://codeforces.com/problemset/problem/241/E)（官方 CF 2600，独立估 2400）。给 DAG 的边赋权，使所有首尾路径等长。关键是先弄清哪些边才受这个条件约束。[分析](difference-constraints-exercises.md#cf241e)

## 超级源的最短路是逐点最大解

超级源的零权边相当于加上 $x_i\le 0$，其中 $x_s=0$。对任何满足这些上界的可行解 $x$，沿 $s$ 到 $i$ 的最短路径相加，得到 $x_i\le d_i$。同时 $d$ 本身可行。所以在 $x_i\le0$ 的条件下，**同一组最短距离同时取到了每个坐标的最大值**，也就最大化了坐标和。若人为加入的只是这组零权边，原系统的可行性不变，因为任何原解都可以整体下移到非正。

更一般地，源边 $s\xrightarrow{b_i}i$ 表示 $x_i\le b_i$。若带这些上界的系统有解，且每个点都从 $s$ 可达，最短距离就是带这些上界的逐点最大解。这也解释了为什么初始化会改变距离的含义。纯差分约束允许任意整体平移，原系统的单个坐标和坐标和都没有有限的极值。

进阶练习 [AGC056 C 01 Balanced](https://atcoder.jp/contests/agc056/tasks/agc056_c)（估 CF 2400）。在若干平衡区间的约束下求字典序最小的二进制串。练习从前缀量的逐点最优恢复字典序最优。[分析](difference-constraints-exercises.md#agc056-c)

零权超级源还有一个性质。设 $d$ 为它求出的解，必有 $\max_i d_i=0$，否则还能整体上移一点并保持所有坐标非正，与逐点最大矛盾。把任意原解 $x$ 整体平移成 $y_i=x_i-\max_j x_j$，则 $y_i\le d_i$。于是

$$
\max_i x_i-\min_i x_i=-\min_i y_i\ge -\min_i d_i=\max_i d_i-\min_i d_i.
$$

因此 $d$ 还**最小化全体变量的极差**。把它整体平移到最小坐标为 $0$，会得到原系统在 $x_i\ge0$ 条件下最小的最大坐标。逐点最小是另一个目标。例如只有 $x_2\le x_1-2$，且 $x_3$ 不受约束时，零权超级源求出 $(0,-2,0)$，平移后是 $(2,0,2)$。真正的非负逐点最小解是 $(2,0,0)$。

### 求逐点最小解

若约束写成 $x_v\ge x_u+w$，且题目给出下界 $x_i\ge a_i$，令 $y_i=-x_i$。原约束变成 $y_v\le y_u-w$，源边 $s\xrightarrow{-a_i}i$ 表示 $y_i\le -a_i$。对 $y$ 求逐点最大解再取反，就得到了 $x$ 的**逐点最小解**。也可以直接在下界图上求最长路，松弛时抬高下界，矛盾对应正权环。

例如 $x_1\ge1,\ x_2\ge1,\ x_3\ge1$，还有 $x_2\ge x_1+2,\ x_3\ge x_2+1$。从 $x_1\ge1$ 依次推出 $x_2\ge3,\ x_3\ge4$，而 $(1,3,4)$ 同时满足全部约束，所以最小总和为 $8$。把最后一条下界改成上界 $x_3\le x_2+1$，它就不再强迫 $x_3$ 增大，最小解变为 $(1,3,1)$。

练习 [P3275 糖果](https://www.luogu.com.cn/problem/P3275)（估 CF 1900）。先推出最小总和为何由逐点最小解取得，再利用边权只有 $0,1$ 写出能处理加强数据的做法。[分析](difference-constraints-exercises.md#p3275)

## 最大化指定的变量差

设原约束图已有解。要最大化 $x_b-x_a$，固定 $x_a=0$，从原图的 $a$ 出发跑最短路。记距离为 $\delta(a,b)$。若 $b$ 可达，路径累加给出 $x_b-x_a\le\delta(a,b)$。还需要证明上界能取到，尤其是有点从 $a$ 不可达时。

要让 $x_b-x_a\ge T$，就加边 $b\xrightarrow{-T}a$。原图无负环，新出现的负环一定用到这条边。删掉这条边，剩下的就是从 $a$ 到 $b$ 的路径。若 $b$ 可达，令 $T=\delta(a,b)$，每条这样的路径和新边组成的环权都非负，系统仍然有解，所以上界恰好取到。若 $b$ 不可达，新边无法形成环，任意大的 $T$ 都可行，因此差值无上界。同理，交换 $a,b$ 就能得到下界 $-\delta(b,a)$，不可达时无下界。

两次最短路回答不同的问题。超级源能看到全图，用于判断所有约束是否同时可行。从 $a$ 出发的距离给出相对于 $a$ 的最紧上界。直接用超级源求出的 $d_b-d_a$，得到的是那组可行解中的一个差值。

### 例题 Layout

奶牛按编号从左到右站立，允许同位置。给定部分距离的上界和下界，求第 $1$ 头与第 $N$ 头奶牛能相距多远。样例给出 $x_3-x_1\le10$、$x_4-x_2\le20$、$x_3-x_2\ge3$。[POJ 3169 原题存档](https://web.stanford.edu/class/cs97si/probs/3169.htm)

从 $1$ 到 $4$ 的路径 $1\xrightarrow{10}3\xrightarrow{-3}2\xrightarrow{20}4$ 给出上界 $27$。位置 $(0,7,10,27)$ 满足顺序和三个约束，取到了上界。顺序本身还给出 $x_i-x_{i+1}\le0$，所以要建边 $i+1\xrightarrow{0}i$。这些边不能省。超级源的零权边只用于判全局可行，样例中用它求出的一个解是 $(-3,-3,0,0)$，两端相差仅 $3$。

练习 [Layout，洛谷 P4878 / POJ 3169](https://www.luogu.com.cn/problem/P4878)（估 CF 1700）。实现完整题目，区分无解、无上界和有有限最大值三种输出。[分析](difference-constraints-exercises.md#poj-3169)

进阶练习 [ABC216 G 01Sequence](https://atcoder.jp/contests/abc216/tasks/abc216_g)（估 CF 2100）。满足区间内 `1` 的个数下界，并构造 `1` 最少的序列。练习把总数写成指定差值，并选一种使边权非负的前缀量。[分析](difference-constraints-exercises.md#abc216-g)

进阶练习 [CF Gym 103934 A The army of Thutmose III](https://codeforces.com/gym/103934/problem/A)（估 CF 2500）。每栋建筑至少要在施工期间被访问一次，求最小的单栋访问次数上限。练习离散化和二分可行性，最后从一组整数势恢复日期。[分析](difference-constraints-exercises.md#cf-gym-103934-a)

## 参考

- [MIT 6.046J Lecture 7，§7.2 Difference Constraints](https://live.ocw.mit.edu/courses/6-046j-design-and-analysis-of-algorithms-spring-2012/a9e76885a78c729f2375e14830caebf2_MIT6_046JS12_lec07.pdf#page=5)
