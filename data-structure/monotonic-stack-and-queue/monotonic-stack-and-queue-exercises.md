# 单调栈与单调队列检测题

每组 3 题，按估算难度递增。完整输入输出与样例见原题。

## 单调栈

1. [ABC372 D · Buildings](https://atcoder.jp/contests/abc372/tasks/abc372_d)（约 1500）

   一排楼的高度各不相同。对每栋楼 $i$，数出右侧有多少栋楼 $j$ 满足：两栋楼之间没有比 $j$ 更高的楼。$n\le 2\times 10^5$。
2. [ABC407 F · Sums of Sliding Window Maximum](https://atcoder.jp/contests/abc407/tasks/abc407_f)（约 2200）

   给定一个可含重复值的非负整数数组。对每个长度 $k=1,\ldots,n$，把所有长度为 $k$ 的连续子数组的最大值加起来。$n\le 2\times 10^5$。
3. [CF2160 G1 · Inverse Minimum Partition](https://codeforces.com/contest/2160/problem/G1)（约 2500）

   将正整数数组切成若干非空连续段。每段的代价是末项除以段内最小值，再向上取整。求最小总代价。所有测试的 $n$ 之和不超过 $4\times 10^5$，$a_i\le 10^{18}$。

## 单调队列

1. [CSES 1644 · Maximum Subarray Sum II](https://cses.fi/problemset/task/1644/)（约 1500）

   给定一个可含负数的整数数组，在长度介于 $a$ 和 $b$ 之间（含端点）的连续子数组中，求最大的元素和。$1\le a\le b\le n\le 2\times 10^5$。
2. [CF372 C · Watching Fireworks is Fun](https://codeforces.com/problemset/problem/372/C)（约 2000）

   街道有 $n$ 个位置，起点任选，每单位时间最多走 $d$ 格。第 $i$ 场烟花在时刻 $t_i$ 从位置 $a_i$ 发射，此时你在 $x$ 就获得 $b_i-|a_i-x|$ 的快乐值。求总快乐值最大值。烟花可以同时发射，$n\le 150000$，场数 $m\le 300$。
3. [CF487 B · Strip](https://codeforces.com/problemset/problem/487/B)（约 2100）

   将数组切成若干连续段。每段至少有 $l$ 个元素，且段内最大值与最小值之差至多为 $s$。求最少段数，无解输出 $-1$。$n\le 10^5$。
