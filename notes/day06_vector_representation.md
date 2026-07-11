# Day 6 Notes: Vector Representation

## 阅读范围

本次阅读范围是论文 *Changing Base without Losing Space* 的 Section 4，重点理解作者如何使用 information carrier lemma 构造最优空间的向量表示。

主要内容包括：

- 向量表示问题与信息论空间下界；
- 将若干向量元素组合成 information carriers；
- Section 4.1 的顺序构造；
- 顺序构造的查询、更新与冗余分析；
- 顺序构造中预计算常量过多的问题；
- Section 4.2 的隐式二叉树表示；
- 如何将预计算常量减少到 $O(\log n)$；
- 最终空间界的精确化。

这一部分承接 Day 5：information carrier lemma 给出了局部保存 spill 的一般方法，Section 4 则将该引理组织成一个完整的数据结构，从而证明论文的第一个主要结果。

------

## 内容总结

论文考虑一个长度为 $n$ 的向量：

$$
A[1\ldots n],
$$

其中每个元素来自大小为 $\Sigma$ 的有限字母表。该向量包含的信息量是：

$$
n\log_2\Sigma,
$$

因此任何表示至少需要：

$$
\left\lceil n\log_2\Sigma\right\rceil
$$

bits。

将整个向量看成一个 $\Sigma$ 进制大整数，可以达到最优空间，但读取或修改一个元素可能需要处理整个向量。逐个元素独立编码虽然支持局部访问，却会因为向上取整产生线性冗余。

作者使用 information carrier 技术同时解决这两个问题：先把若干元素组成一个较大的 carrier，再通过 spill 在相邻节点之间传递尚未写入的信息。顺序构造已经能够获得接近最优的空间和常数时间局部访问，但需要过多的预计算常量。最终的树形构造让同层节点共享参数，将预计算常量数量降到 $O(\log n)$。

------

## 论文具体解析

## 向量分块

将连续的若干向量元素组合成一个较大的值：

$$
x_i\in[X].
$$

论文选择：

$$
X=\Theta(n^2).
$$

因此每个 carrier 大约包含：

$$
\Theta(\log_{\Sigma} n)
$$

个原始元素，carrier 的总数为：

$$
N=
O\left(
\frac{n}{\log_{\Sigma}n}
\right).
$$

这样选择 $X$ 的原因来自 information carrier lemma。每次使用该引理产生的冗余是：

$$
O\left(\frac{1}{\sqrt X}\right).
$$

当 $X=\Theta(n^2)$ 时，每一步冗余变为：

$$
O\left(\frac{1}{n}\right).
$$

经过 $N$ 次转换后，累计冗余为：

$$
N\cdot O\left(\frac{1}{n}\right)
=
O\left(\frac{1}{\log n}\right)
=
o(1).
$$

因此，把原始元素组合成足够大的 carrier，可以让所有局部 rounding loss 的总和小于一个 bit。

------

## 顺序构造：A First Attempt

设分块后的 carrier 为：

$$
x_1,x_2,\ldots,x_N.
$$

算法从一个空 spill $y_0$ 开始。第 $i$ 步将当前 carrier $x_i$ 和旧 spill $y_{i-1}$ 输入 information carrier lemma：

$$
(x_i,y_{i-1})
\longmapsto
(m_i,y_i).
$$

其中：

- $m_i$ 是写入二进制内存的内容；
- $y_i$ 是传递给下一步的新 spill。

整个过程可以表示为：

```text
y0 + x1 -> m1 + y1
y1 + x2 -> m2 + y2
y2 + x3 -> m3 + y3
...
```

最后将剩余的 spill $y_N$ 单独保存。

------

## 顺序构造的空间分析

最终使用的空间由所有 memory contents 和最后一个 spill 组成：
$$
\left\lceil\log_2Y_N\right\rceil
+
\sum_{i=1}^{N}M_i.
$$
减去原始 carriers 包含的信息量 $N\log_2X$，得到冗余：
$$
\left\lceil\log_2Y_N\right\rceil
+
\sum_iM_i
-
N\log_2X.
$$
将其按每次 information carrier 转换重新分组：
$$
\leq
1+
\sum_i
\left(
M_i+\log_2Y_i-\log_2Y_{i-1}-\log_2X
\right).
$$
中间 spill 的空间项会相互抵消，只剩最后一个 spill 的取整损失，以及每次转换产生的局部冗余。因此：
$$
\text{redundancy}
\leq
1+
N\cdot O\left(\frac{1}{\sqrt X}\right)
=
1+o(1).
$$
这里的关键是：spill 并不是每一步都独立保存，而是作为下一步的输入继续传递，所以不会产生线性累积的取整浪费。

------

## 顺序构造中的查询

若要读取属于 carrier $x_j$ 的某个元素：

1. 从下一步的 memory content $m_{j+1}$ 中恢复 spill $y_j$；
2. 使用 $m_j$ 和 $y_j$ 恢复 carrier $x_j$；
3. 从 $x_j$ 中提取目标向量元素。

查询不需要从 $y_0$ 开始顺序解码所有 spill。

这是因为 information carrier lemma 保证：旧 spill 可以仅从当前 memory content 中恢复。因此查询只访问常数个 memory regions，时间复杂度为：
$$
O(1).
$$

------

## 顺序构造中的更新

若修改 carrier $x_j$：

1. 使用新的 $x_j$ 重新执行第 $j$ 步；
2. 得到新的 memory content $m_j$ 和新的 spill $y_j$；
3. 因为 $y_j$ 发生变化，重新执行第 $j+1$ 步；
4. 第 $j+1$ 步产生的输出 spill $y_{j+1}$ 不需要继续变化。

因此一次修改只需要重算相邻的常数个编码步骤，更新时间仍为：
$$
O(1).
$$

------

## 顺序构造的问题

顺序构造已经基本实现了：

- 接近最优的空间；
- $O(1)$ 查询；
- $O(1)$ 更新。

但它需要：
$$
O\left(\frac{n}{\log n}\right)
$$
组预计算常量。

原因是每一步接收到的 spill universe $Y_i$ 可能不同，因此每一步使用的：

- memory 长度 $M_i$；
- 新 spill 范围；
- 除法和乘法常量；
- memory offset；

都可能不同。

这不满足论文定理要求的：
$$
O(\log n)
$$
个预计算 word constants。

------

## 隐式二叉树表示

为减少预计算常量，作者将 carriers 放入一棵隐式二叉树中，并按照 heap 方式编号：

```
节点 i 的左孩子：2i
节点 i 的右孩子：2i+1
节点 i 的父节点：floor(i/2)
```

树结构不需要显式存储指针。

每个节点接收两个孩子传来的 spills，将它们组合成一个值 $y_i$，再使用节点自身的 carrier $x_i$ 作为 information carrier：
$$
(x_i,y_i)
\longmapsto
(m_i,s_i).
$$
其中：

- $m_i$ 写入该节点对应的 memory region；
- $s_i$ 继续传递给父节点。

可以将节点处理过程理解为：

```
left spill  ---\
                -> combine -> yi
right spill ---/               \
                                  + xi -> mi + spill to parent
```

------

## 树结构中的查询与更新

若要读取节点 $v$ 中的 carrier：

1. 从父节点的 memory content 中恢复节点 $v$ 传出的 spill；
2. 读取节点 $v$ 自身的 memory content；
3. 使用二者恢复 $x_v$；
4. 从 $x_v$ 中提取目标元素。

虽然编码时 spill 会沿树向上传递，但查询时不需要从根节点递归向下解码。一次查询只涉及节点及其父节点附近的常数个 memory regions，因此仍然是：
$$
O(1).
$$
修改 $x_v$ 时，只需要重新编码节点 $v$ 和它的父节点。节点 $v$ 的新 spill 被父节点吸收后，不会继续沿整条祖先链传播，因此更新也是：
$$
O(1).
$$

------

## 为什么只需要 $O(\log n)$ 个预计算常量

树中同一层的节点，其子树形状只有三种情况：

1. 一段节点拥有高度为 $k$ 的完整二叉子树；
2. 至多一个节点拥有不完整子树；
3. 剩余节点拥有高度为 $k-1$ 的完整二叉子树。

同一层、同一类别的节点具有相同的子树规模，因此：

- 接收到的 spill ranges 相同；
- 使用的 information carrier 参数相同；
- memory content 的长度和定位方式相同。

所以每一层只需要常数种预计算参数。

树的高度为：
$$
O(\log N)=O(\log n),
$$
因此总预计算常量数量为：
$$
O(\log n).
$$

------

## 最终空间界

information carrier lemma 产生的累计冗余为：
$$
o(1).
$$
保存最终根 spill 时，向上取整最多可能再损失一个 bit，因此直接分析得到：
$$
\left\lceil
n\log_2\Sigma+o(1)
\right\rceil.
$$
论文进一步增大 carrier 的规模，将冗余压低到：
$$
O(n^{-c})
$$
并使用关于 logarithmic forms 的数论结果，保证对于固定字母表大小 $\Sigma$，该误差不足以跨越下一个整数边界。

最终空间可以精确达到：
$$
\left\lceil n\log_2\Sigma\right\rceil.
$$

------

## 最终结论

论文最终证明，在 Word RAM 模型中，可以使用：
$$
\left\lceil n\log_2\Sigma\right\rceil
$$
bits 表示长度为 $n$、字母表大小为 $\Sigma$ 的向量，同时支持：
$$
O(1)
$$
时间读取和修改任意元素，并且只需要：
$$
O(\log n)
$$
个预计算 word constants。

该构造最重要的地方是，它没有在最优空间和局部访问之间做传统意义上的折中。information carrier 负责消除局部取整浪费，而树结构负责让局部编码参数可以大规模共享。