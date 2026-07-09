# Day 4 Notes: SOLE Encoding

## 阅读范围

本次阅读范围主要是论文 *Changing Base without Losing Space* 的 Section 2，重点围绕 SOLE prefix-free encoding 展开。

具体来说，本次阅读关注以下主题：

- SOLE encoding 的问题设定；
- 为什么加入 `eof` 后，字母表从 `[B]` 扩大到 `[B + 1]`；
- Basic SOLE 对 `n`、`b`、`B` 的假设；
- Pass 1 如何把 `[B + 1]^2` 转换成 `[B - 4i] × [B + 4i + 4]`；
- Pass 2 如何通过重新分组把数据转换回 `[B]^2`；
- SOLE 的终止处理；
- SOLE 的局部性性质；
- 论文中 `n = 3, B = 32, M = (5, 7, 31)` 的例子；
- SOLE 与后续 information carriers 的关系。

---

## 内容总结

Day 4 主要理解了论文中的第一个具体构造：SOLE encoding。它出现在 Section 2，是后续 information carrier lemma 的 warm-up。Day 3 已经说明，online prefix-free encoding 的关键不是简单给每个 block 加一个结束标记 bit，而是把加入 `eof` 后的字母表扩大问题看成 base conversion 问题。原始 block 来自 `[B]`，加入 `eof` 后变成来自 `[B + 1]`。SOLE 的目标就是把这个 `[B + 1]` 字母表上的流重新编码回普通的 `[B]` blocks，同时避免每个 block 浪费一个完整 bit。

Basic SOLE 的核心做法分成两个 pass。Pass 1 把相邻两个 `[B + 1]` 符号看成一个大数，然后分解到一个略短区间和一个略长区间中，即：

```text
[B + 1]^2 -> [B - 4i] × [B + 4i + 4]
```

其中短区间放在奇数位置，长区间放在偶数位置，所以叫 Short-Odd Long-Even。Pass 2 再错位重新分组，把相邻的长区间和短区间合并。由于：

```text
(B + 4i)(B - 4i) < B^2
```

重新分组后的每一对都能写成两个普通 `[B]` blocks。

SOLE 最重要的意义不只是节省空间，而是展示了局部可逆性。论文的 Property 3 说明，常数个输入 blocks 决定常数个输出 blocks，反过来常数个输出 blocks 也能恢复常数个输入 blocks。这种局部性是全文的核心思想之一。Section 3 的 information carriers 正是从 SOLE 中抽象出来：当某个值处在非 2 的幂大小的区间中，不能直接无损写入二进制时，可以借助附近的信息块把它局部地保存下来，同时产生新的 spill。

------

## 论文具体解析

## SOLE Encoding 的问题设定

设每个输入 block 有 `b` bits，令：

```text
B = 2^b
[B] = {0, 1, ..., B - 1}
```

原始输入 block 是 `[B]` 中的元素。为了让在线流具有 prefix-free 性质，需要加入一个特殊的结束符号：

```text
eof
```

加入 `eof` 后，每个符号的字母表大小从：

```text
B
```

变成：

```text
B + 1
```

如果直接为每个 block 加一个额外 bit，会产生线性冗余。SOLE 的目标是更精细地完成换基：

```text
把来自 [B + 1] 的流重新编码成来自 [B] 的 blocks。
```

Basic SOLE 假设：

```text
n 为奇数
b >= 2 log₂ n + 1
B = 2^b >= 2n²
```

在这些条件下，算法把 `n` 个输入 blocks 加上 `eof` 后编码为 `n + 2` 个 `[B]` blocks。

------

## Pass 1：Short-Odd Long-Even 转换

Pass 1 处理位置：

```text
(2i + 1, 2i + 2)
```

上的一对符号。每个符号来自 `[B + 1]`，所以一对符号可以看成一个属于：

```text
[B + 1]^2
```

的大数。

SOLE 将这个大数分解到：

```text
[B - 4i] × [B + 4i + 4]
```

中。第一个区间略短，第二个区间略长。短区间放在奇数位置，长区间放在偶数位置，所以称为 Short-Odd Long-Even。

具体做法是：把两个输入符号合成一个大数 `z`，然后对 `B - 4i` 做除法：

```text
z = quotient · (B - 4i) + remainder
```

其中：

```text
remainder ∈ [B - 4i]
quotient  ∈ [B + 4i + 4]
```

这一转换合法的关键条件是：

```text
(B + 1)^2 <= (B - 4i)(B + 4i + 4)
```

这说明目标区间的总容量足够容纳原来的 `[B + 1]^2` 个可能输入，因此可以通过商和余数唯一恢复原来的 pair。

------

## Pass 2：重新分组回到 `[B]^2`

Pass 1 后，位置上的区间大致交替变成：

```text
[B], [B + 4], [B - 4], [B + 8], [B - 8], ...
```

Pass 2 不再按照原始 pair 分组，而是错位重新分组，处理形如：

```text
[B + 4i] × [B - 4i]
```

的一对值。

由于：

```text
(B + 4i)(B - 4i) = B² - 16i² < B²
```

所以这一对值的总可能数小于 `B²`。因此它可以被编码成两个普通的 `[B]` blocks，即：

```text
[B + 4i] × [B - 4i] -> [B]^2
```

这一步是 SOLE 的关键技巧：Pass 1 先制造交替的 short / long ranges，Pass 2 再通过错位分组让 long 和 short 抵消，使每一对重新落回标准的 `[B]^2`。

------

## 终止处理

Basic SOLE 假设 `n` 是奇数，因此 `eof` 会出现在偶数位置：

```text
n + 1 = 2i
```

在 Pass 1 之后，算法还会人工加入一个 `0`，它被看作属于：

```text
[B - 4i - 4]
```

的值。这样可以补齐最后一组，使 Pass 2 能正常输出最后的 blocks。

解码时，一旦恢复出 `eof`，解码器就立即停止。因此编码本身具有 prefix-free 性质。论文用 Property 3 保证解码器不会为了发现 `eof` 而读过编码文件的结尾。

------

## 局部性性质

SOLE 的重要性质是局部性。论文给出 Property 3：

```text
Output blocks 2i and 2i + 1 can be computed from input blocks 2i - 1, ..., 2i + 2.

Input blocks 2i + 1 and 2i + 2 can be decoded from output blocks 2i, ..., 2i + 3.
```

这说明 SOLE 不是普通的全局压缩算法。它的编码和解码都只依赖常数个相邻 blocks。因此，修改某个输入 block 时，只需要重新计算附近常数个输出 blocks。

这一点和全文主线密切相关。论文最终要实现的是既保持最优空间，又支持局部访问和局部修改。SOLE 在 prefix-free encoding 的简化场景中先展示了这种局部换基的可能性。

------

## 论文中的小例子

论文给出的例子是：

```text
n = 3
B = 32
input = (5, 7, 31)
```

此时每个普通 block 属于 `[32]`，加入 `eof` 后字母表变为 `[33]`，其中：

```text
eof = 32
```

所以扩展后的输入为：

```text
(5, 7, 31, 32, 0)
```

第一对 `(5, 7)` 来自 `[33] × [33]`。先合成大数：

```text
z = 7 · 33 + 5 = 236
```

然后分解到 `[32] × [36]`：

```text
236 = 7 · 32 + 12
```

所以第一对变为：

```text
(12, 7)
```

第二对 `(31, 32)` 同样来自 `[33] × [33]`。合成大数：

```text
z = 32 · 33 + 31 = 1087
```

此时要分解到 `[28] × [40]`：

```text
1087 = 38 · 28 + 23
```

所以第二对变为：

```text
(23, 38)
```

Pass 1 后得到：

```text
(12, 7, 23, 38, 0)
```

Pass 2 重新分组：

```text
12
(7, 23)
(38, 0)
```

其中：

```text
23 · 36 + 7 = 835 = 26 · 32 + 3
```

所以得到两个 `[32]` blocks：

```text
(3, 26)
```

最后一组：

```text
0 · 40 + 38 = 38 = 1 · 32 + 6
```

得到：

```text
(6, 1)
```

因此最终输出为：

```text
(12, 3, 26, 6, 1)
```

这个例子展示了 SOLE 如何把 3 个输入 blocks 加上 `eof` 编码成 5 个普通 `[B]` blocks。

------

## Section 2.3 的推广与实际意义

Section 2.3 讨论了 SOLE 的一些性质和改进。

首先，由于 SOLE 具有局部性，随机访问、修改、追加和删除末尾 blocks 都可以通过访问常数个相邻 blocks 完成。这说明 SOLE 不只是一个前缀编码方案，也体现了局部数据结构的思想。

其次，从实际角度看，如果取：

```text
b = 32 或 b = 64
```

算法中的乘法和除法可以通过 word-level arithmetic 高效实现。论文指出，基本 SOLE 在大块上运行时，额外开销远小于朴素的每 block 一个 marker bit 方法。

最后，论文还提到可以通过更紧的终止处理把浪费从最多 3 个 blocks 降到更接近最优的程度。这部分不是后续主线的核心，但说明 SOLE 的基本思想还可以继续优化。