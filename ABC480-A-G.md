# AtCoder Beginner Contest 480：A–G 思路整理

比赛日期：2026-10-17。以下依据原题与官方解说整理，侧重建模、关键性质、正确性与实现边界；各题附可独立编译的 GNU C++17、O2 参考实现，沿用你的代码风格。不包含个人提交评审。D 的元素、块和查询位置统一使用 0-index。

| 题目 | 核心方法 | 时间复杂度 |
|---|---|---|
| A — ADjacent | 坐标与相邻关系 | $O(1)$ |
| B — Invalid Vote | 完全匹配优先，再统计一处失配 | $O(NML)$ |
| C — Self-Stopping Shower | 排序、延后操作、批量填补空档 | $O(N\log N)$ |
| D — Concatenation of Prefixes | 两层前缀和、块边界二分 | $O(N+Q\log N)$ |
| E — Product-Bounded Arithmetic Sequences | 枚举首项与长度、二分公差 | 见本题分析 |
| F — Cake Cutting | 划分建模、子集 DP、消去余数维度 | $O(N2^N)$ |
| G — Zonal Score Minimization | 划分 DP、两个单调栈、区间加与区间最小值 | $O(N\log N)$ |

## A — ADjacent

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_a) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27253)

**目标。** 在 $2\times2$ 网格中，判断 A 和 D 是否共边。

设它们的位置分别为 $(r_A,c_A)$、$(r_D,c_D)$。相邻当且仅当

$$
|r_A-r_D|+|c_A-c_D|=1.
$$

这是曼哈顿距离的直接应用。由于本题只有两行两列，且 A、D 不在同一格，也可以判断“同行或同列”。后一个条件只适用于这里的特殊大小，不能原样推广到一般网格。

**实现取舍。** 把两行拼成长度为 4 的字符串，用 `find` 定位 A、D，再用 `i / 2`、`i % 2` 还原行列，足够简洁。不必为四个格子建立图，也不必列举字符排列。

时间、空间均为 $O(1)$。

### 参考实现

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    string s, t;
    cin >> s >> t;
    s += t;
    int u = s.find('A'), v = s.find('D');
    cout << (u / 2 == v / 2 || u % 2 == v % 2 ? "Yes" : "No") << "\n";
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## B — Invalid Vote

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_b) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27254)

**目标。** 每张选票先尝试完全匹配；只有不存在完全匹配时，才允许改一个字符。能由此匹配的候选人恰好一个，选票才有效。

对每张选票 $T$：

1. 若存在 $S_i=T$，直接给 $i$ 计票。
2. 否则遍历所有与 $T$ 等长的名字，统计恰有一个位置不同的候选人。
3. 仅当这样的候选人有一个时计票。

**关键在优先级。** 完全匹配的候选人不会因为附近还有其他名字而丢票。例如名字为 `a`、`b`，选票为 `a`，仍然投给 `a`。

“改一个字符”是替换，不包括插入、删除，因此长度不同可以直接跳过。等长时，只要计算汉明距离；发现两处失配后就不必继续比较。

**实现取舍。** 只需要保存“匹配数量”和“最后一个匹配者”，不必构造候选列表。这里 $N,M\le100$、名字长度 $L\le10$，直接枚举最清晰，不需要 Trie 或字符串哈希。

时间 $O(NML)$。逐张读取选票时，额外空间为 $O(NL)$。

### 参考实现

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    int n, m;
    cin >> n >> m;
    vector<string> a(n);
    for (auto& s : a) {
        cin >> s;
    }

    vector<int> ans(n);
    while (m--) {
        string s;
        cin >> s;
        auto it = find(a.begin(), a.end(), s);
        if (it != a.end()) {
            ans[it - a.begin()]++;
            continue;
        }

        int c = 0, id = -1;
        for (int i = 0; i < n; i++) {
            if (a[i].size() != s.size()) {
                continue;
            }
            int d = 0;
            for (int j = 0; j < (int) s.size(); j++) {
                d += a[i][j] != s[j];
            }
            if (d == 1) {
                c++;
                id = i;
            }
        }
        if (c == 1) {
            ans[id]++;
        }
    }

    for (auto x : ans) {
        cout << x << "\n";
    }
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## C — Self-Stopping Shower

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_c) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27255)

**模型。** 每次在时刻 $t$ 按按钮，提供区间 $[t,t+K]$ 的热水。所有人的按钮效果等价，后按的按钮只会把结束时刻推迟，因此可看成这些区间的并。目标是覆盖整个 $[0,T]$，只计算自己添加的区间数。

### 为什么可以尽量晚按

如果当前热水能持续到 $r$，你却在某个 $s<r$ 提前按，那么把这次操作推迟到 $r$ 不会留下空档，结束时刻还会从 $s+K$ 推迟到 $r+K$。

如果在需要自己按之前，别人已按下按钮，则先利用这次免费操作即可。因此存在一个最优策略：**只在即将断水时自己按。**

### 批量处理空档

将 $A$ 排序。维护最近一次已处理的免费按键带来的结束时刻 $r$；初始令 $r=0$，表示尚未获得正长度的覆盖。

处理一个 $a<T$：

- 若 $a\le r$，到 $a$ 前不用额外按。
- 若 $a>r$，长度为 $a-r$ 的空档需要

$$
q=\left\lceil\frac{a-r}{K}\right\rceil
$$

次操作，在 $r,r+K,\ldots,r+(q-1)K$ 按即可。
- 免费按钮在 $a$ 按下后，更新 $r=a+K$。

这里最后一次自己按的结束时刻 $r+qK<a+K$，所以免费按钮确实覆盖得更远；无需把额外状态延续到下一段。处理完所有 $a<T$ 后，再补上最后的空档：

$$
\left\lceil\frac{\max(0,T-r)}{K}\right\rceil.
$$

初始空档也适用这个公式：若第一个免费按钮不在 0，最优策略会从时刻 0 开始按。若免费按钮在 0，则不需自己启动。

**边界。** 区间两端都有热水，恰好在结束时刻续按合法。但时间连续，不能等到下一个整数时刻。$A_i\ge T$ 的免费按钮可以忽略；重复时刻不影响算法。大空档要用整除批量计数，不能逐次模拟。

排序后扫描，时间 $O(N\log N)$，存储 $O(N)$。计数、时间及上取整中间量采用 64 位整数更稳妥。

官方解说使用坐标压缩与差分来找未覆盖区间；上面的“延后操作”证明给出了直接排序扫描的做法，状态更少。

### 参考实现

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    int n;
    long k, t;
    cin >> n >> k >> t;
    vector<long> a(n);
    for (auto& x : a) {
        cin >> x;
    }
    sort(a.begin(), a.end());

    auto need = [&](long x) {
        return (max(0LL, x) + k - 1) / k;
    };

    long ans = 0, r = 0;
    for (auto x : a) {
        if (x >= t) {
            break;
        }
        ans += need(x - r);
        r = x + k;
    }
    cout << ans + need(t - r) << "\n";
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## D — Concatenation of Prefixes

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_d) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27257)

**模型。** 将原数组记为 $a_0,\ldots,a_{n-1}$，第 $i$ 个块就是 $(a_0,\ldots,a_i)$。不应展开 $B$，因为其长度可能超过 $5\times10^9$。

### 全部按 0-index 定义

定义：

$$
p_i=\sum_{j=0}^{i}a_j,\qquad
s_i=\sum_{j=0}^{i}p_j,\qquad
\operatorname{from}_i=\frac{i(i+1)}2.
$$

`p[i]` 是原数组闭区间 $[0,i]$ 的和，`s[i]` 是前缀和数组闭区间 $[0,i]$ 的和，`from[i]` 是第 $i$ 个块在 $B$ 中的起始下标。三者都只使用下标 $[0,n)$。

令 $F(x)$ 表示 $B$ 的闭区间 $[0,x]$ 的和，并约定 $F(-1)=0$。

求 $F(x)$ 时，找到最后一个满足 $\operatorname{from}_i\le x$ 的块。这是 $x$ 所在的块，前面的完整块贡献 $s_{i-1}$；当前块取到块内下标 $x-\operatorname{from}_i$，贡献 $p_{x-\operatorname{from}_i}$。因此

$$
\boxed{F(x)=(i=0\ ?\ 0:s_{i-1})+p_{x-\operatorname{from}_i}}.
$$

输入的 $L,R$ 减一，得到 0-index 的闭区间 $[l,r]$；查询答案是

$$
F(r)-F(l-1).
$$

### 为什么这一写法仍然整洁

只需要在前缀函数中处理空前缀 `x < 0`，并在没有前置完整块时将其贡献取为 0。其余情况统一为“前面所有完整块 + 当前块前缀”，无需区分同块或跨块。恰好位于块起点、块末尾、整个 $B$ 的最后一项，都使用同一公式。

实现中连续调用两次 `partial_sum`：第一次原地把输入变成 `p`，第二次由 `p` 得到 `s`。`upper_bound(from.begin(), from.end(), x) - 1` 定位所在块。这里不会出现块编号 $n$，因为二分的是每块的起点。

**类型边界。** $l,r$ 与块起点必须使用 64 位；总和绝对值最多为 $10^8\cdot n(n+1)/2\le5.00005\times10^{17}$，64 位足够。块编号仍可使用 `int`。计算三角数时，以 `1LL` 提前提升乘法类型。

时间 $O(n+Q\log n)$，空间 $O(n)$。若用求根公式定位块，需要处理浮点误差；这里二分更直接可靠。

### 参考实现

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    int n, q;
    cin >> n >> q;
    vector<long> p(n), s(n), from(n);
    for (auto& x : p) {
        cin >> x;
    }
    partial_sum(p.begin(), p.end(), p.begin());
    partial_sum(p.begin(), p.end(), s.begin());
    for (int i = 0; i < n; i++) {
        from[i] = 1LL * i * (i + 1) / 2;
    }

    auto sum = [&](long x) -> long {
        if (x < 0) {
            return 0;
        }
        int i = upper_bound(from.begin(), from.end(), x) - from.begin() - 1;
        return (i ? s[i - 1] : 0) + p[x - from[i]];
    };

    while (q--) {
        long l, r;
        cin >> l >> r;
        l--, r--;
        cout << sum(r) - sum(l - 1) << "\n";
    }
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## E — Product-Bounded Arithmetic Sequences

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_e) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27258)

**模型。** 一个序列由首项 $a\ge1$、公差 $d\ge1$、长度 $k\ge3$ 唯一确定，要求

$$
P(a,d,k)=\prod_{i=0}^{k-1}(a+id)\le N.
$$

答案很大，不适合逐个枚举合法序列；应枚举部分参数，一次性统计剩余参数。

### 两个小维度与一个单调维度

- 因为 $a+id\ge i+1$，所以 $P\ge k!$。$20!>10^{18}$，故只需 $3\le k\le19$。
- 因为 $P\ge a^k$，所以固定长度时只需 $a\le\lfloor N^{1/k}\rfloor$。长度 3 时最多约 $10^6$，更长时迅速缩小。
- 固定 $a,k$，$P$ 随 $d$ 严格递增，因此合法公差恰好为 $1,2,\ldots,D$。

枚举 $k,a$，二分最大的合法公差 $D$，给答案加 $D$。每个序列只有一组 $(a,d,k)$，所以既不遗漏，也不重复。

实际可以用 $P(a,1,k)\le N$ 决定首项循环是否继续：最小公差都不可行时，更大的公差同样不可行。由此避免对整数根的浮点估算。

### 不溢出的判定

不直接乘出整个乘积，而从 $v=N$ 开始，依次执行

$$
v\leftarrow\left\lfloor\frac{v}{a+id}\right\rfloor.
$$

正整数除数满足

$$
\left\lfloor\frac{\lfloor N/x\rfloor}{y}\right\rfloor
=\left\lfloor\frac{N}{xy}\right\rfloor,
$$

因此最后 $v>0$ 当且仅当原乘积不超过 $N$。中途变成 0 就能立即返回失败。构造 $a+id$ 时，也要先把乘法提升到 64 位。

**二分边界。** $d=10^9$ 一定不可行，因为仅前三项已有

$$
a(a+d)(a+2d)\ge(d+1)(2d+1)>10^{18}.
$$

若已经确认 $d=1$ 可行，就可以维护 `l = 1` 可行、`r = 1'000'000'000` 不可行。

### 实际代价

令 $A_k$ 为长度 $k$ 时可行首项的数量，则 $A_k\le\lfloor N^{1/k}\rfloor$。上述实现的时间上界为

$$
O\!\left(\log N\sum_{k=3}^{19}k\lfloor N^{1/k}\rfloor\right).
$$

主要开销来自长度 3 的约百万个首项，每个二分约 30 次、每次最多三次除法。无需把所有长度都套用百万级首项上界。额外空间 $O(1)$。

**可迁移认识。** 参数有三个，不代表必须三重枚举；先用增长速度压小两个参数，再利用单调性把第三个参数整段计数。

### 参考实现

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    long n;
    cin >> n;

    auto good = [&](long a, long d, int k) {
        long x = n;
        for (int i = 0; i < k; i++) {
            x /= a + i * d;
            if (!x) {
                return false;
            }
        }
        return true;
    };

    long ans = 0;
    for (int k = 3; good(1, 1, k); k++) {
        for (long a = 1; good(a, 1, k); a++) {
            long l = 1, r = 1'000'000'000;
            while (r - l > 1) {
                long m = (l + r) / 2;
                if (good(a, m, k)) {
                    l = m;
                } else {
                    r = m;
                }
            }
            ans += l;
        }
    }
    cout << ans << "\n";
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## F — Cake Cutting

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_f) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27259)

**模型。** 设总大小为 $S$，每人应获得 $K=S/M$。切割位置可以是实数，但最优值可以只由原始蛋糕的子集和决定。

定义 $f$：把所有原始蛋糕划分成若干个非空组，使每组总大小都是 $K$ 的正整数倍，最多能分成多少组。

关键结论是

$$
\boxed{\text{最少切割次数}=M-f}.
$$

### 为什么每多一组就少切一次

**构造上界。** 一组总大小为 $qK$。把组内蛋糕依次排成一条长度为 $qK$ 的长条，在 $K,2K,\ldots,(q-1)K$ 处划分；若分界线落在蛋糕内部就切开，落在原有边界上则不用切。

因此这一组至多切 $q-1$ 次。各组的 $q$ 之和是 $M$，合计至多 $M-f$ 次。

**证明下界。** 对任意切割、分配方案建立二分图：一侧是 $N$ 个原始蛋糕，另一侧是 $M$ 个人；每块最终蛋糕在“其来源”和“其接收者”之间连一条边，允许重边。

若切了 $c$ 次，就产生 $N+c$ 块，即 $N+c$ 条边。设图有 $h$ 个连通分量，连通图至少有“顶点数减一”条边，所以

$$
N+c\ge N+M-h,\qquad c\ge M-h.
$$

每个连通分量包含的原始蛋糕总大小，恰好等于该分量中人数乘 $K$；于是这些原始蛋糕组成一个合法分组。因此 $h\le f$，得到 $c\ge M-f$。

上下界一致，公式成立。这也解释了为什么不能只看每块蛋糕对 $K$ 的商：不同蛋糕的余量可以互相配合。

### 如何用 $O(N2^N)$ 求最多分组数

把每组内的蛋糕排在一起，就得到一种排列。反过来，排列中的前缀和每次成为 $K$ 的倍数，都可以结束当前组。

设 $w[mask]$ 为所选蛋糕的总大小，$dp[mask]$ 为排列这些蛋糕时，能得到的最多完整组数，允许末尾留一个尚未完整的组。

当前未完成组的大小模 $K$，必定等于

$$
w[mask]\bmod K.
$$

已经结束的组都贡献 $K$ 的倍数，因此**余数由集合唯一决定，不需要额外一维 DP**。

考虑最后加入的蛋糕 $i$：

$$
dp[mask]
=\max_{i\in mask}dp[mask\setminus\{i\}]
+[w[mask]\bmod K=0],\qquad dp[0]=0.
$$

其中方括号表示条件成立为 1，否则为 0。只对非空集合使用转移，所以不会把空前缀算成一组。

加入最后一块后，若总和模 $K$ 为 0，就能结束当前组；否则保持未完成。这覆盖了所有排列，也自然覆盖了所有合法分组。全集的总和是 $MK$，末尾一定完整，答案为

$$
M-dp[(1\ll N)-1].
$$

**实现边界。** $N\le20$，子集数量约百万；$w$ 与 $K$ 使用 64 位，$dp$ 使用 `int` 即可。预处理子集和后枚举集合中存在的位，不必重复计算余数，也不要枚举所有子集划分的子集，那会退化到 $O(3^N)$。

时间 $O(N2^N)$，空间 $O(2^N)$。每块蛋糕恰好为 $K$ 时答案为 0；只有一块蛋糕时答案为 $M-1$。

### 参考实现

`s &= s - 1` 每次清除最低的一个置位；仅当 `s` 非零时调用 `__builtin_ctz(s)`，用于枚举集合中实际存在的元素。

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

void solve() {
    int n, m;
    cin >> n >> m;
    vector<long> a(n);
    for (auto& x : a) {
        cin >> x;
    }
    long k = accumulate(a.begin(), a.end(), 0LL) / m;

    vector<long> w(1 << n);
    vector<int> dp(1 << n);
    for (int mask = 1; mask < (1 << n); mask++) {
        int i = __builtin_ctz(mask);
        w[mask] = w[mask ^ (1 << i)] + a[i];
        for (int s = mask; s; s &= s - 1) {
            int j = __builtin_ctz(s);
            dp[mask] = max(dp[mask], dp[mask ^ (1 << j)]);
        }
        dp[mask] += w[mask] % k == 0;
    }
    cout << m - dp.back() << "\n";
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## G — Zonal Score Minimization

[原题](https://atcoder.jp/contests/abc480/tasks/abc480_g) · [官方解说](https://atcoder.jp/contests/abc480/editorial/27256)

**目标。** 将数组划分成若干连续段，每段长度至少为 2，最小化各段的“最大值减最小值”之和。

### 先写出完整的划分 DP

设 $dp_i$ 为前 $i$ 个数的最优值：

$$
dp_0=0,\qquad dp_1=+\infty,
$$

$$
dp_i=\min_{0\le j\le i-2}
\left(dp_j+\max_{j<t\le i}A_t-\min_{j<t\le i}A_t\right).
$$

$j$ 表示最后一段前面有多少个元素。长度至少为 2，正是 $j\le i-2$ 的来源。直接枚举所有 $j$ 是 $O(N^2)$。

### 不加速某一次转移，而是维护所有候选

固定当前右端点 $i$，在线段树下标 $j$ 上维护

$$
V_j=dp_j+\max(A_{j+1},\ldots,A_i)-\min(A_{j+1},\ldots,A_i).
$$

只需支持三种操作：

- 单点赋值：加入新候选 $j=i-1$，初值为 $dp_{i-1}$，因为单元素区间的极差为 0。
- 区间加：右端点右移后，修正某些候选的最大、最小值。
- 区间最小值：查询 $j\in[0,i-2]$，得到 $dp_i$。

新候选虽然暂时对应长度 1，但本轮查询将它排除；下一轮起，它才有资格参与答案。$dp_1=+\infty$ 又排除了“前面留下一个单元素段”的方案。

### 为什么单调栈恰好匹配区间加

对固定的右端点，随着 $j$ 从小到大移动，区间逐渐缩短，所以最大值不增、最小值不减。相同最大值或最小值的 $j$，形成连续区间。

最大值栈保存 `(value, left)`：该最大值控制的候选区间左端点为 `left`，右端点由下一个栈元素的左端点确定。栈中 `value` 严格递减。

加入 $x=A_i$ 时，栈尾所有 $value\le x$ 的区间都会改用 $x$。对一个原值为 $v$ 的区间，给线段树加上 $x-v$。把受影响的区间合并，再压入 $x$。

最小值栈对称处理：栈中 `value` 严格递增，弹出所有 $value\ge x$ 的区间，对它们加上 $v-x$。

注意两类增量都非负：最大值变大、最小值变小，都会使已有区间的极差增大。

### 精确的更新顺序

以下伪代码中的候选区间使用闭区间：

```text
单点赋值 V[i-1] = dp[i-1]

left = i-1
while 最大值栈非空且栈顶 value <= A[i]:
    弹出 (value, l)
    对 V[l .. left-1] 加 A[i] - value
    left = l
压入 (A[i], left)

left = i-1
while 最小值栈非空且栈顶 value >= A[i]:
    弹出 (value, l)
    对 V[l .. left-1] 加 value - A[i]
    left = l
压入 (A[i], left)

若 i >= 2:
    dp[i] = min(V[0 .. i-2])
```

两个栈都必须重新从 `left = i-1` 开始。每次弹栈后再令 `left = l`，保证更新的是原栈顶控制的那一段，不会与此前弹出的区间重叠。

### 正确性与复杂度

单点赋值正确初始化了新区间；两个单调栈恰好找出所有最大值、最小值发生变化的候选，并给出精确增量，因此线段树始终维护 $V_j$ 的定义。最后按合法 $j$ 范围取最小值，就是原始 DP 转移。

每个位置在每个栈中只入栈、出栈一次，所以总共只有 $O(N)$ 次区间加。每次线段树操作 $O(\log N)$，总时间 $O(N\log N)$，空间 $O(N)$。

**实现边界。** 整个数组不分割也是合法方案，所以答案不超过整个数组的极差，即 $10^9-1$。DP 和线段树仍建议使用 64 位，以便安全设置无穷值并执行区间加；相等值统一合并，减少栈中冗余区间。线段树只需“区间加 + 区间最小值 + 单点赋值”，不需要 Segment Tree Beats。

**可迁移认识。** 转移里的区间最值不好直接查询时，可以改问：右端点增加一个元素后，哪些决策的代价改变了？如果改变范围能由单调栈批量描述，就能把 $O(N)$ 个决策的更新压缩成少量区间操作。

### 参考实现

代码中的数组下标为 0-index，`dp[k]` 仍表示前 $k$ 项的最优值。处理 `a[i]` 时查询候选 $[0,i-1]$，计算 `dp[i+1]`；线段树公开接口使用闭区间，内部使用半开区间。

```cpp
#include <bits/stdc++.h>
#define long long long

using namespace std;

constexpr long INF = 1LL << 60;

struct tag {
    long add = 0;
};

struct node {
    long x = INF;
    void apply(const tag& v) {
        x += v.add;
    }
    friend node operator + (const node& a, const node& b) {
        return {min(a.x, b.x)};
    }
};

struct segment_tree {
    int n;
    vector<node> a;
    vector<tag> lazy;

    segment_tree(int n) : n(n), a(n * 4), lazy(n * 4) {
    }

    void apply(int p, const tag& v) {
        a[p].apply(v);
        lazy[p].add += v.add;
    }

    void spread(int p) {
        if (lazy[p].add) {
            apply(p * 2, lazy[p]);
            apply(p * 2 + 1, lazy[p]);
            lazy[p] = {};
        }
    }

    void set(int p, int l, int r, int x, const node& v) {
        if (r - l == 1) {
            a[p] = v;
            lazy[p] = {};
            return;
        }
        spread(p);
        int m = (l + r) / 2;
        if (x < m) {
            set(p * 2, l, m, x, v);
        } else {
            set(p * 2 + 1, m, r, x, v);
        }
        a[p] = a[p * 2] + a[p * 2 + 1];
    }

    void update(int p, int l, int r, int x, int y, const tag& v) {
        if (r <= x || y <= l) {
            return;
        }
        if (x <= l && r <= y) {
            apply(p, v);
            return;
        }
        spread(p);
        int m = (l + r) / 2;
        update(p * 2, l, m, x, y, v);
        update(p * 2 + 1, m, r, x, y, v);
        a[p] = a[p * 2] + a[p * 2 + 1];
    }

    node query(int p, int l, int r, int x, int y) {
        if (r <= x || y <= l) {
            return {};
        }
        if (x <= l && r <= y) {
            return a[p];
        }
        spread(p);
        int m = (l + r) / 2;
        return query(p * 2, l, m, x, y) + query(p * 2 + 1, m, r, x, y);
    }

    void set(int x, long v) {
        set(1, 0, n, x, {v});
    }

    void update(int l, int r, long v) {
        if (l <= r) {
            update(1, 0, n, l, r + 1, {v});
        }
    }

    node query(int l, int r) {
        return query(1, 0, n, l, r + 1);
    }
};

void solve() {
    int n;
    cin >> n;
    vector<long> a(n), dp(n + 1, INF);
    for (auto& x : a) {
        cin >> x;
    }
    dp[0] = 0;

    segment_tree t(n);
    vector<pair<long, int>> mx, mn;
    for (int i = 0; i < n; i++) {
        t.set(i, dp[i]);

        int l = i;
        while (!mx.empty() && mx.back().first <= a[i]) {
            auto [v, from] = mx.back();
            mx.pop_back();
            t.update(from, l - 1, a[i] - v);
            l = from;
        }
        mx.emplace_back(a[i], l);

        l = i;
        while (!mn.empty() && mn.back().first >= a[i]) {
            auto [v, from] = mn.back();
            mn.pop_back();
            t.update(from, l - 1, v - a[i]);
            l = from;
        }
        mn.emplace_back(a[i], l);

        if (i > 0) {
            dp[i + 1] = t.query(0, i - 1).x;
        }
    }
    cout << dp[n] << "\n";
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(nullptr);
    solve();
}
```

## 核对说明

以上推导与 A–G 官方解说逐题对照；F 的下界另外用“原始蛋糕—接收者”二分图给出了证明。本文七份完整实现均通过 GNU C++17、O2 编译和全部官方样例，并完成以下核对：

- A：穷举 A、B、C、D 的全部 24 种排列。
- B：100 组随机选票数据，与按规则独立分类的结果比较。
- C：100 组小规模随机数据，与枚举所有整数时刻按键集合比较；另检查长达 $10^9$ 的空档。
- D：100 组随机数组，与显式构造 $B$ 的暴力比较，覆盖所有块末尾、负数和单元素数组；另检查 $n=10^5$、超过 `int` 上限的查询位置及最大和。
- E：$N=1\ldots200$ 与暴力枚举比较，并核对官方大样例 $10^{18}\mapsto1051148114007$。
- F：100 组小规模随机数据，与直接枚举合法子集划分比较；另检查 $n=20$、总和超过 `int` 上限和单蛋糕分给 100 人。
- G：完整的单调栈与懒标记线段树代码通过 500 组随机数据对拍，对照 $O(n^2)$ DP；另检查 $n=2\times10^5$ 的全相等、交替高低和严格递增数组。

原题及官方解说链接已附在各题开头。以上为本地验证，未进行 AtCoder 在线提交。
