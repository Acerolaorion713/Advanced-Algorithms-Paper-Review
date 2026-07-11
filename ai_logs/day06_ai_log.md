# Day 6 AI Log

## 今日任务背景

今天的阅读内容集中在论文 *Changing Base without Losing Space* 的 Section 4，目标是理解作者如何使用 information carrier lemma 构造最优空间的向量表示。

本日使用 AI 的目的主要是：

- 梳理 Section 4 的整体技术路线；
- 理解为什么要将原始向量元素组合成较大的 carriers；
- 推导顺序构造的累计冗余；
- 理解顺序构造中的常数时间查询与更新；
- 判断顺序构造为什么仍不足以证明 Theorem 1；
- 理解隐式二叉树表示如何减少预计算常量；
- 核验最终空间界、查询复杂度和更新复杂度。

本日 AI 输出只作为辅助理解材料，写入 notes 或 review 前需要对照论文 Section 4 核验。

------

## AI 使用记录

## Prompt 1：Section 4 的整体技术路线

**我提出的问题：**

> 请根据论文 *Changing Base without Losing Space* 的 Section 4，解释作者如何从 information carrier lemma 得到最优空间的向量表示。重点区分 Section 4.1 的顺序构造和 Section 4.2 的树形构造。

**使用 AI 的目的：**

理解 Section 4 在全文中的位置，并为 `notes/day06_vector_representation.md` 的“内容总结”部分整理技术主线。

**AI 回答摘要：**

AI 将 Section 4 的技术路线概括为：

```text
向量分块
    ↓
把每个大块作为 information carrier
    ↓
使用顺序构造传递 spill
    ↓
获得低冗余和常数时间局部访问
    ↓
发现预计算常量数量过多
    ↓
改用隐式二叉树组织 carriers
    ↓
同层同类节点共享参数
    ↓
得到 O(log n) 个预计算常量
```

顺序构造已经解决了空间冗余和局部访问问题，树形构造主要解决参数无法共享的问题。

**采纳内容：**

采纳了整体技术路线，并写入：

- `notes/day06_vector_representation.md` 的“内容总结”；
- `notes/day06_vector_representation.md` 的“顺序构造的问题”；
- `notes/day06_vector_representation.md` 的“隐式二叉树表示”。

**未采纳或暂不使用内容：**

AI 曾将树结构描述为“把顺序链并行化”。这一说法容易使人误以为论文主要在优化并行时间，因此未采用。树结构的核心作用是让节点按层和子树类型共享编码参数。

**人工核验情况：**

已对照论文 Section 4.1 和 Section 4.2。论文先给出 sequential first attempt，再明确指出其需要过多预计算常量，随后使用 tree representation 将常量数量减少到 $O(\log n)$。

**对应修改文件：**

- `notes/day06_vector_representation.md`
- 后续可用于 `review/sections/05_vector_representation.tex`

------

## Prompt 2：向量分块与 carrier universe

**我提出的问题：**

> 请解释为什么论文要把若干向量元素组合成一个 carrier，并推导为什么 carrier universe 取 $X=\Theta(n^2)$，以及 carrier 数量为什么是 $O(n/\log_{\Sigma}n)$。

**使用 AI 的目的：**

核验 Section 4 中的分块参数，以及该参数与 information carrier lemma 冗余界之间的关系。

**AI 回答摘要：**

设每个原始元素来自大小为 $\Sigma$ 的字母表。若一个 carrier 包含 $t$ 个元素，则其取值范围大小约为：
$$
X=\Sigma^t.
$$
令：
$$
X=\Theta(n^2),
$$
可以得到：
$$
t=\Theta(\log_{\Sigma}n).
$$
因此长度为 $n$ 的向量被分成：
$$
N=
O\left(
\frac{n}{\log_{\Sigma}n}
\right)
$$
个 carriers。

information carrier lemma 每次转换的冗余是：
$$
O\left(\frac{1}{\sqrt X}\right).
$$
当 $X=\Theta(n^2)$ 时，每步冗余为：
$$
O\left(\frac{1}{n}\right).
$$
经过 $N$ 步后总冗余为：
$$
N\cdot O\left(\frac{1}{n}\right)
=
O\left(\frac{1}{\log n}\right)
=
o(1).
$$
**采纳内容：**

采纳了分块规模和累计冗余的推导，写入 notes 的“向量分块”部分。

**未采纳或暂不使用内容：**

AI 曾直接把 carrier 数量写成 $\Theta(n/\log n)$。为了保留字母表大小的影响，notes 中先写为：
$$
O\left(\frac{n}{\log_{\Sigma}n}\right),
$$
在固定 $\Sigma$ 的情况下再理解为 $O(n/\log n)$。

**人工核验情况：**

已对照论文 Section 4.1。论文将元素分组到大小为 $O(\log_{\Sigma}n)$ 的 blocks 中，使每个 block 可以解释为来自 $X=\Theta(n^2)$ 的值，并由此得到每步 $O(1/n)$ 的冗余。

**对应修改文件：**

- `notes/day06_vector_representation.md`

------

## Prompt 3：顺序构造与冗余分析

**我提出的问题：**

> 请详细解释 Section 4.1 的 sequential construction，并推导为什么总冗余可以写成 $1+N\cdot O(1/\sqrt X)$。中间 spill 的空间项为什么会相互抵消？

**使用 AI 的目的：**

辅助理解论文使用的 chain-rule / telescoping 分析。

**AI 回答摘要：**

设 carriers 为：
$$
x_1,x_2,\ldots,x_N.
$$
第 $i$ 步执行：
$$
(x_i,y_{i-1})
\longmapsto
(m_i,y_i).
$$
最终存储所有 $m_i$ 和最后一个 spill $y_N$。总冗余可以写为：
$$
\left\lceil\log_2Y_N\right\rceil
+
\sum_iM_i
-
N\log_2X.
$$
将每个中间 spill 的对数范围加入并减去后，可以改写为：
$$
1+
\sum_i
\left(
M_i+\log_2Y_i-\log_2Y_{i-1}-\log_2X
\right).
$$
每个括号正好对应一次 information carrier 转换的冗余，而中间的：
$$
+\log_2Y_i
$$
会与下一步中的：
$$
-\log_2Y_i
$$
抵消。因此只剩每步局部冗余和最后 spill 的向上取整损失。

**采纳内容：**

采纳了 telescoping 的解释和公式重组方式，写入 notes 的“顺序构造的空间分析”部分。

**未采纳或暂不使用内容：**

AI 曾将最后的 $1$ 解释为 information carrier lemma 本身产生的一 bit 冗余。该说法不准确。最后的 $1$ 主要来自保存最终 spill 时的向上取整。

**人工核验情况：**

已对照论文 Section 4.1 中的正式不等式。论文确实利用 redundancy 的 chain rule，将总冗余限制为：
$$
1+N\cdot O\left(\frac{1}{\sqrt X}\right).
$$
**对应修改文件：**

- `notes/day06_vector_representation.md`
- 后续可用于 `review/sections/05_vector_representation.tex`

------

## Prompt 4：顺序构造中的查询与更新

**我提出的问题：**

> 请说明 sequential construction 中如何在常数时间恢复某个 carrier $x_j$，以及修改 $x_j$ 时为什么只需要重新计算相邻两个步骤。

**使用 AI 的目的：**

核验顺序构造的局部访问性质，并理解 Day 5 中“旧 spill 可以从 memory content 单独恢复”的实际作用。

**AI 回答摘要：**

要读取 $x_j$：

1. 从下一步的 memory content $m_{j+1}$ 中恢复 spill $y_j$；
2. 使用当前 memory content $m_j$ 和 spill $y_j$ 恢复 $x_j$；
3. 从 $x_j$ 中提取目标原始元素。

因此，不需要从第一个 spill 开始顺序解码。

修改 $x_j$ 时，需要重新执行第 $j$ 步。该步骤产生的 spill $y_j$ 可能变化，因此还要重新执行第 $j+1$ 步。第 $j+1$ 步重新吸收新的 $y_j$ 后，其输出 spill $y_{j+1}$ 保持不变，所以变化不会继续传播。

**采纳内容：**

采纳了查询和更新过程，写入：

- `顺序构造中的查询`
- `顺序构造中的更新`

**未采纳或暂不使用内容：**

AI 最初将查询描述为只读取一个 memory block。严格来说，恢复一个 carrier 需要当前步骤和相邻步骤的常数个 memory contents，因此 notes 中使用“常数个 memory regions”，而不是“一个 block”。

**人工核验情况：**

已对照论文 Section 4.1。论文明确说明，读取 $x_j$ 时先从第 $j+1$ 步的 memory 中恢复 spill，再结合第 $j$ 步的 memory 恢复 $x_j$；更新只需要重算步骤 $j$ 和 $j+1$。

**对应修改文件：**

- `notes/day06_vector_representation.md`

------

## Prompt 5：顺序构造为什么仍不够

**我提出的问题：**

> Section 4.1 的顺序构造已经有低冗余和常数时间查询、更新，为什么它仍然不能直接证明 Theorem 1？

**使用 AI 的目的：**

区分顺序构造的真正缺陷与空间、查询复杂度问题。

**AI 回答摘要：**

顺序构造的问题不是冗余过大，也不是访问时间过慢，而是每一步的 spill universe $Y_i$ 可能不同。

因此每一步可能需要独立保存：

- information carrier lemma 的参数；
- memory bit 长度；
- division 和 multiplication 常量；
- memory 中的起始位置。

一共有：
$$
N=
O\left(\frac{n}{\log n}\right)
$$
个步骤，所以可能需要同样数量的预计算常量。这超过了 Theorem 1 要求的：
$$
O(\log n).
$$
**采纳内容：**

采纳了“顺序构造的唯一主要缺陷是预计算常量过多”的判断，写入 notes 的“顺序构造的问题”部分。

**未采纳或暂不使用内容：**

AI 曾将顺序构造称为“不正确的构造”。该说法不准确。顺序构造本身是可逆且局部的，只是没有满足定理对预计算常量数量的要求。

**人工核验情况：**

已对照论文 Section 4.1 结尾。论文明确指出，每个 $Y_j$ 可能不同，memory bits 数量也会变化，因此需要 $N=O(n/\log n)$ 个预计算常量。

**对应修改文件：**

- `notes/day06_vector_representation.md`

------

## Prompt 6：隐式二叉树表示

**我提出的问题：**

> 请解释 Section 4.2 的 implicit binary-tree representation。每个节点如何处理两个 child spills，查询和更新为什么仍然是 $O(1)$？

**使用 AI 的目的：**

理解最终向量数据结构的组织方式。

**AI 回答摘要：**

carriers 按照 heap 编号放入隐式二叉树：

```
left child  = 2i
right child = 2i + 1
parent      = floor(i / 2)
```

每个节点接收左右孩子传来的 spills，将它们组合成一个值 $y_i$，再使用自身 carrier $x_i$ 执行：
$$
(x_i,y_i)
\longmapsto
(m_i,s_i).
$$
其中 $m_i$ 写入内存，$s_i$ 传给父节点。

读取节点 $v$ 的 carrier 时，从父节点的 memory 中恢复 $v$ 的 spill，再结合节点自身的 memory 恢复 $x_v$。因此只访问节点和父节点附近的常数个位置。

修改 $x_v$ 时，只需要重算节点 $v$ 和其父节点。节点产生的新 spill 在父节点处被吸收，不需要继续沿祖先链传播。

**采纳内容：**

采纳了树形编码、查询和更新的解释，写入：

- `隐式二叉树表示`
- `树结构中的查询与更新`

**未采纳或暂不使用内容：**

AI 曾认为查询需要从根节点沿路径走到目标节点，因此复杂度为 $O(\log n)$。这一理解与论文的局部恢复机制不符，未采用。

**人工核验情况：**

已对照论文 Section 4.2。论文明确说明，虽然 spills 在构造时向上传递，但查询时没有递归：通过父节点的 memory 恢复目标节点的 spill，再结合目标节点自身的 memory 恢复其 carrier。

**对应修改文件：**

- `notes/day06_vector_representation.md`
- 后续可用于 `review/sections/05_vector_representation.tex`

------

## Prompt 7：为什么只需要 $O(\log n)$ 个常量

**我提出的问题：**

> 为什么 tree representation 中每一层的节点只需要常数种参数？论文所说的三类节点分别是什么？

**使用 AI 的目的：**

核验树结构如何解决顺序构造中的参数数量问题。

**AI 回答摘要：**

在某一层上，节点的子树只会出现三种类型：

1. 一段节点拥有高度为 $k$ 的完整二叉子树；
2. 至多一个节点拥有不完整子树；
3. 剩余节点拥有高度为 $k-1$ 的完整二叉子树。

同一层、同一类型的节点具有相同的子树规模，因此孩子传来的 spill ranges 相同，information carrier lemma 使用的参数也相同。

每层只需要常数组参数，而树高为：
$$
O(\log n).
$$
因此预计算常量总数为：
$$
O(\log n).
$$
**采纳内容：**

采纳了同层三类节点和参数共享的解释，写入 notes 的“为什么只需要 $O(\log n)$ 个预计算常量”部分。

**未采纳或暂不使用内容：**

AI 曾说同一层所有节点都使用完全相同的参数。该说法忽略了不完整子树以及高度相差一的完整子树，因此改为“每层只有常数种参数”。

**人工核验情况：**

已对照论文 Section 4.2。论文确实将每层节点分为完整高度 $k$、至多一个不完整节点、完整高度 $k-1$ 三类。

**对应修改文件：**

- `notes/day06_vector_representation.md`

------

## Prompt 8：最终空间界

**我提出的问题：**

> 请解释 Section 4.2 的 last-bit 问题。为什么直接构造可能得到 $\lceil n\log_2\Sigma+o(1)\rceil$，作者又如何得到精确的 $\lceil n\log_2\Sigma\rceil$？

**使用 AI 的目的：**

理解 Theorem 1 中精确最优空间界的最后一步。

**AI 回答摘要：**

information carrier 的总 rounding loss 可以压缩到：
$$
o(1).
$$
但最终 spill 必须使用整数个 bits 保存，因此直接分析得到：
$$
\left\lceil
n\log_2\Sigma+o(1)
\right\rceil.
$$
该值理论上可能比：
$$
\left\lceil n\log_2\Sigma\right\rceil
$$
多一 bit。

论文进一步增大 carrier universe，并使用更宽的整数运算，把误差降低为：
$$
O(n^{-c}).
$$
再利用 linear forms in logarithms 的结果，说明对于固定 $\Sigma$，$n\log_2\Sigma$ 与最近整数之间存在 inverse-polynomial 级别的间隔。选择足够大的 $c$ 后，额外误差不会跨过下一个整数边界。

**采纳内容：**

采纳了 last-bit refinement 的整体解释，写入 notes 的“最终空间界”部分。

**未采纳或暂不使用内容：**

AI 曾尝试展开 Baker--Wüstholz 定理的完整数论证明。该内容超出 Day 6 的重点，因此 notes 中只保留其在空间精确化中的作用，具体文献和证明留到 related work 或 proof checklist 阶段处理。

**人工核验情况：**

已对照论文 Section 4.2 的 “The last bit” 部分。论文确实通过减小误差并引用 linear forms in logarithms 的结果，将空间界精确到：
$$
\left\lceil n\log_2\Sigma\right\rceil.
$$
**对应修改文件：**

- `notes/day06_vector_representation.md`
- 后续可用于 `review/sections/05_vector_representation.tex`
- 相关文献留待 `review/references.bib` 后续补充