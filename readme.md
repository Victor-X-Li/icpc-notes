# ICPC Notes

## 专题讲解

每个小专题单独建目录，讲解与对应的 `-exercises.md` 放在同一目录。以下 15 个专题均有配套习题分析。

| 分类 | 小专题与讲解 | 习题分析 |
| --- | --- | --- |
| data structure | [01 Trie 与二进制建模](data-structure/binary-trie/binary-trie.md) | [exercise](data-structure/binary-trie/binary-trie-exercises.md) |
| data structure | [笛卡尔树与单调栈](data-structure/cartesian-tree-and-monotonic-stack/cartesian-tree-and-monotonic-stack.md) | [exercise](data-structure/cartesian-tree-and-monotonic-stack/cartesian-tree-and-monotonic-stack-exercises.md) |
| graph and tree | [Dijkstra 进阶与最短路树](graph-and-tree/dijkstra-and-shortest-path-trees/dijkstra-and-shortest-path-trees.md) | [exercise](graph-and-tree/dijkstra-and-shortest-path-trees/dijkstra-and-shortest-path-trees-exercises.md) |
| graph and tree | [DFS 树、Tarjan 与圆方树](graph-and-tree/dfs-tarjan-and-block-cut-trees/dfs-tarjan-and-block-cut-trees.md) | [exercise](graph-and-tree/dfs-tarjan-and-block-cut-trees/dfs-tarjan-and-block-cut-trees-exercises.md) |
| graph and tree | [优化建图与隐式图](graph-and-tree/graph-modeling/graph-modeling.md) | [exercise](graph-and-tree/graph-modeling/graph-modeling-exercises.md) |
| graph and tree | [LCA 的进阶性质与应用](graph-and-tree/lca-properties/lca-properties.md) | [exercise](graph-and-tree/lca-properties/lca-properties-exercises.md) |
| graph and tree | [启发式合并与 DSU on Tree](graph-and-tree/small-to-large-and-dsu-on-tree/small-to-large-and-dsu-on-tree.md) | [exercise](graph-and-tree/small-to-large-and-dsu-on-tree/small-to-large-and-dsu-on-tree-exercises.md) |
| mathematics | [数学期望](mathematics/expected-value/expected-value.md) | [exercise](mathematics/expected-value/expected-value-exercises.md) |
| mathematics | [容斥原理](mathematics/inclusion-exclusion/inclusion-exclusion.md) | [exercise](mathematics/inclusion-exclusion/inclusion-exclusion-exercises.md) |
| mathematics | [ICPC 数论：欧拉筛、CRT 与常用定理](mathematics/number-theory/number-theory.md) | [exercise](mathematics/number-theory/number-theory-exercises.md) |
| mathematics | [组合数学](mathematics/combinatorics/combinatorics.md) | [exercise](mathematics/combinatorics/combinatorics-exercises.md) |
| mathematics | [组合数学 + DP](mathematics/combinatorial-dp/combinatorial-dp.md) | [exercise](mathematics/combinatorial-dp/combinatorial-dp-exercises.md) |
| mathematics | [线性基与异或空间（含最值、排名参考代码与团队比赛回练）](mathematics/xor-basis/xor-basis.md) | [exercise](mathematics/xor-basis/xor-basis-exercises.md) |
| string | [KMP、Border 理论与失配树（1-index；自动机为赛后附录）](string/kmp-and-border-theory/kmp-and-border-theory.md) | [exercise](string/kmp-and-border-theory/kmp-and-border-theory-exercises.md) |
| string | [随机数与字符串哈希](string/randomness-and-string-hashing/randomness-and-string-hashing.md) | [exercise](string/randomness-and-string-hashing/randomness-and-string-hashing-exercises.md) |

其他知识点放在 [misc/](misc/readme.md)，目前尚无独立专题。线性基按数学原理归入 mathematics；启发式合并与 DSU on Tree 归入 graph and tree。

## 比赛复盘

所有 AtCoder、Codeforces、QOJ 等比赛的复盘统一放在 [contest/](contest/readme.md)。比赛文章已含完整分析，不另建重复的 exercise 文件。

| 比赛 | 复盘内容 |
| --- | --- |
| [2024 CCPC 哈尔滨站 / 第三届 Universal Cup 哈尔滨站](contest/harbin-2024-review.md) | 估算难度 ≤2500：C、M、G、K、J、A、B、L |
| [2025 ICPC 成都区域赛 / 第四届 Universal Cup 成都站](contest/chengdu-2025-tutorial.md) | 估算难度 ≤2500 的八题，以及额外提高题 E |
| [AtCoder ABC480](contest/ABC480-A-G.md) | A–G 题解与参考实现 |
| [AtCoder ABC481](contest/abc481-review.md) | E、F、G 完整分析 |
| [Codeforces Round 1129 (Div. 2)，CF2277](contest/cf-2277-review.md) | C、E、F 完整分析 |

## 文件与同步约定

专题目录使用 `data-structure/`、`graph-and-tree/`、`mathematics/`、`string/`、`misc/`；每个小专题沿用语义化英文目录名与原有文章文件名。共享图片集中放在根目录 `asset/`：专题文章使用 `../../asset/`，比赛文章使用 `../asset/` 相对链接，可离线阅读。原 ZIP 中的文章、配图与 Benq 的 PDF 附件均保留。

后续讲解和比赛复盘继续按此目录结构添加，并同步目录与习题分析；不因增加文章而改写已认可的旧讲解。

仓库地址：https://github.com/Victor-X-Li/icpc-notes

首次同步：`git clone https://github.com/Victor-X-Li/icpc-notes.git`；以后在仓库内运行 `git pull --ff-only`。若本地有自己的修改，先提交或暂存再同步。

## 训练复盘

- [2026-10-27：KMP 收尾与线性基入门](misc/training-recaps/2026-10-27.md)
