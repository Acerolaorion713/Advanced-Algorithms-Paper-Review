# Day 7 Notes: Online Prefix-Free Encoding

## 阅读范围

本次主要阅读论文 *Changing Base without Losing Space* 的 Section 5（PDF 第 13 页，论文正文页码 13），并辅助回顾 Section 1.2（PDF 第 3--5 页）与 Section 3（PDF 第 10--11 页）。今日重点是未知最终长度的 binary stream 如何被在线转换为 prefix-free code，以及 information carrier lemma 如何把带有特殊终止符的非二进制 alphabet 局部转换回 binary。

本日对应的正式 review 文件是：

```text
review/sections/06_online_prefix_free_encoding.tex
```

这一部分承接前两日：Day 5 给出 information carrier lemma；Day 6 将其用于最优空间向量表示；Day 7 则将同一局部原语用于最终长度未知的在线 prefix-free coding。这是论文的第二个主要结果。

------

## 内容总结

给定在线到达的 $n$-bit stream，最终长度 $n$ 事先未知。论文构造一个 prefix-free encoding，使总长度为：

$$
n+\log_2 n+O(\log\log n),
$$

编码和解码只需 $O(\log n)$ bits 工作空间，并在 Word RAM 上对每个 word/block 使用 $O(1)$ 时间。

构造先把输入划分为缓慢增长的 blocks。每个完整 block 有两个 $k_i$-bit components，可写成 $x_i\in[2^{k_i}]^2$。最后一个不完整 block 单独用小规模 Elias encoding 处理。Pass 1 在 block $N-2$ 中提前嵌入 $\mathrm{eof}$，并把被替换的第二 component 移到尾部；Pass 2 顺序应用 information carrier lemma，把扩大后的 block alphabet 转成 binary。block 尺寸随已处理输入增长，使每步 $O(2^{-k_i})$ 的损失按尺度求和仅为 $O(\log\log n)$。

------

## 论文具体解析

## 在线 Prefix-Free Encoding 的问题设定

Prefix-free 表示任意一个合法 codeword 都不是另一个合法 codeword 的前缀。因此 decoder 不需要额外获得 $n$，也能确定当前消息在哪里结束。

离线方案可以先读完整个消息、得到 $n$，再把长度头写在内容之前。在线 encoder 却不能这样做：它在输出最前面的 bits 时尚不知道最终 $n$，而保存全部输入再回写长度又违反 $O(\log n)$-bit memory 的目标。decoder 同样必须在顺序读取时识别终止位置，不能依赖一个未提供的外部长度。

若给每个固定 block 增加一个 EOF flag，则每个 block 至少损失一个 bit。block 数量随输入线性增长时，总冗余也是线性的。Section 5 的目标正是避免这种 per-block integral rounding。

------

## 为什么传统方案不适合在线场景

Elias code 在已知整数长度时可以给出接近最优的自定界表示，但在线 encoder 无法在消息主体之前输出尚未知的最终长度。另一方面，固定 block 的 EOF bit 虽然容易流式实现，却把很小的字母表扩大成本强制向上取整为一个完整 bit，累计代价过大。

论文不为每个 block 独立保存 flag，而是把“普通值或 $\mathrm{eof}$”视为一个略大的 alphabet，再用 information carriers 将其连续、局部地压回 binary。

------

## Slowly Growing Blocks

当已经看到某个当前规模（论文在局部记号中仍写作 $n$）的输入后，下一个 block 的总长度取为：

$$
2k_i=2\left\lceil\log_2 n\right\rceil.
$$

这里的 $n$ 是决定当前 block 尺寸时已经看到的规模，不是 encoder 事先知道的最终输入长度。第 $i$ 个完整 block 被分成两个 $k_i$-bit components：

$$
x_i=(a_i,b_i)\in[2^{k_i}]^2.
$$

所以 $k_i$ 是每个 component 的位数，block 总长度是 $2k_i$，二者不能混淆。

选择约 $2\log n$ bits 的 block 是为了配合 information carrier lemma。block universe 大约为：

$$
X_i=2^{2k_i}.
$$

引理的单步冗余是 $O(1/\sqrt{X_i})$，因而此处为：

$$
O\left(\frac{1}{\sqrt{2^{2k_i}}}\right)
=O(2^{-k_i}).
$$

随着流增长，$k_i$ 缓慢增大，后续 block 的 rounding loss 迅速减小；固定 block size 不具备这一性质。

------

## 最后一个不完整 Block 的处理

设最终共有 $N$ 个 blocks。前 $N-1$ 个 blocks 完整，第 $N$ 个 block 可能只有 $1$ 到 $2k_N$ bits。论文先以 prefix-free 方式处理前 $N-1$ 个完整 blocks，再在尾部追加第 $N$ 个 block 的 Elias encoding。

这部分只编码最后一个小 block 所需的长度/边界信息，其 overhead 为：

$$
O(\log k_N)=O(\log\log n).
$$

Elias code 没有被用来重新编码整个 $n$-bit stream，也不是先把最终消息长度 $n$ 写到开头；主体流仍由在线的两次转换处理。

------

## Pass 1：提前嵌入 EOF

每个原始完整 block 为：

$$
(a,b)\in[2^{k_i}]^2.
$$

Pass 1 将第二个 component 的 alphabet 扩大一个特殊值：

$$
[2^{k_i}]
\times
\left([2^{k_i}]\cup\{\mathrm{eof}\}\right).
$$

对最终序列的 block $N-2$，变换明确为：

```text
原 block N-2： (a, b)
Pass 1 后：    (a, eof)
被替换的 b：  移到最终特殊尾部，以 plaintext component 保存
```

EOF 不是放在最后一个 block。information carrier 的局部解码有一 block 的依赖：恢复 input block $N-2$ 的 spill，需要读到 output block $N-1$。把 $\mathrm{eof}$ 放在 $N-2$，decoder 在到达普通输出尾部之前就能发现它，然后转入特殊尾部处理。

------

## 三个 Block 的 Lag / Buffer

论文使用一个 $O(\log n)$-bit buffer 引入 3-block lag，相当于让 encoder 提前三个 blocks 得知文件终止。encoder 收到某个 block 时无法立即判断它最终是否会成为 $N-2$，所以不能把最后常数个 blocks 立即输出。

正确做法是缓存最后常数个 blocks。输入结束后，encoder 才确定 $N-2$，把它的第二 component 改成 $\mathrm{eof}$，并按特殊尾部格式输出剩余数据。这不是回头修改已经输出的 bits，而是延迟最后常数个 blocks 的输出。每个 block 只有 $O(\log n)$ bits，因此常数个 blocks 的 buffer 仍为 $O(\log n)$ bits。

------

## Alphabet Expansion 的冗余

原 block universe 大小是 $2^{2k_i}$，扩大后的大小是：

$$
2^{k_i}(2^{k_i}+1).
$$

单 block 的额外信息量为：

$$
\begin{aligned}
&\log_2\left(2^{k_i}(2^{k_i}+1)\right)
-\log_2(2^{2k_i})\\
&=\log_2(1+2^{-k_i})
=O(2^{-k_i}).
\end{aligned}
$$

总和不能只按 block 数粗略相乘。对固定尺度 $K$，论文指出满足 $k_i=K$ 的 blocks 约有：

$$
O\left(\frac{2^K}{K}\right).
$$

每个这样的 block 损失 $O(2^{-K})$，所以该尺度贡献：

$$
O\left(\frac{2^K}{K}\right)O(2^{-K})
=O\left(\frac{1}{K}\right).
$$

从 $K=1$ 求和到 $K=\log_2 n$ 得到调和级数：

$$
\sum_{K=1}^{\log n}O\left(\frac{1}{K}\right)
=O(\log\log n).
$$

------

## Pass 2：通过 Information Carriers 转换为 Binary

Pass 1 后，第 $i$ 个 block 来自扩大后的 alphabet。Pass 2 使用 Day 5 的 information carrier lemma，像 Section 4.1 的顺序 carrier chain 一样依次执行：

$$
(x_i,s_{i-1})\longmapsto(m_i,s_i).
$$

其中 $x_i$ 是当前扩大 alphabet 的 block，$s_{i-1}$ 是旧 spill，$m_i$ 是可以立即写入 binary output 的 memory content，$s_i$ 是传给下一 block 的新 spill。Lemma 4 保证旧 spill 可从当前 memory content 局部恢复，而当前 carrier 可由 memory content 和新 spill 恢复。

当前 carrier domain 的规模是 $X_i=O(2^{2k_i})$，因此每步 carrier redundancy 为 $O(2^{-k_i})$。使用与 alphabet enlargement 相同的分尺度计数，其累计冗余也是 $O(\log\log n)$。每一步仅需常数次 word arithmetic。

------

## Decoder 如何及时发现 EOF

解码停止过程必须结合 EOF 的位置和 carrier 的局部依赖理解：

1. $\mathrm{eof}$ 被编码在 input block $N-2$ 的第二 component；
2. 顺序 carrier 的局部性质使 decoder 在读到 output block $N-1$ 后得到恢复 input block $N-2$ 所需的 spill；
3. decoder 此时恢复整个 input block $N-2$，因此在越过编码尾部前看见 $\mathrm{eof}$；
4. 检测到 $\mathrm{eof}$ 后，decoder 停止普通 carrier 解码；
5. decoder 转入特殊尾部解码，处理第 $N$ 个不完整 block；
6. decoder 读取从 block $N-2$ 移到尾部的第二 component $b$；
7. 把 $b$ 放回 $N-2$，再接上最后 partial block，恢复原 bit stream；
8. 特殊尾部不会被误解为更多普通 blocks。

所以“提前”不是一般性的格式偏好，而是 decoder stopping condition 所要求的具体位置安排。

------

## 总冗余分析

| 冗余来源 | 开销 |
|---|---:|
| early EOF 与 displaced component | $\leq \log_2 n$ |
| alphabet enlargement | $O(\log\log n)$ |
| information carrier redundancy | $O(\log\log n)$ |
| final partial block | $O(\log\log n)$ |
| total | $\log_2 n+O(\log\log n)$ |

主要的 $\log_2 n$ 项来自用 $\mathrm{eof}$ 替换 block $N-2$ 的一个 $k_{N-2}$-bit component，并把该 component 移到尾部；论文给出 $k_{N-2}\leq\log_2 n$。这里 displaced component 是该机制的数据保存方式，不能再作为另一个独立的 $\log n$ 项重复计数。其余三项都是 $O(\log\log n)$。因此总编码长度是：

$$
n+\log_2 n+O(\log\log n).
$$

------

## 工作空间与运行时间

工作空间包括常数个增长 blocks 的 buffer、一个 bounded spill，以及少量 counters 和当前 block 参数。每个对象均占 $O(\log n)$ bits，所以总工作空间是：

$$
O(\log n)\text{ bits}.
$$

每个 block 的两次转换只使用常数次加法、乘法、除法和商余数拆分。在论文的 Word RAM 模型中，每个 word/block 的处理时间为 $O(1)$；整个输入的总时间与处理的 blocks 数量线性相关。特殊结尾仅涉及常数个 blocks。

------

## 与 SOLE Encoding 的关系

SOLE 是固定 block size 和简化条件下的 warm-up。它用 short/long ranges 展示普通 binary blocks 如何携带非二进制 spill，并说明局部转换可以避免逐 block 向上取整。Section 5 不再依赖 SOLE 的特定 parity 结构，而是使用一般的 information carrier lemma，同时加入 slowly growing blocks、未知长度下的 3-block lag 和特殊 termination。

------

## 与 Elias Code 的关系

离线 Elias coding 可以先编码已知的整体长度，接近信息论最优；在线 encoder 却不能在流开始时知道最终 $n$。本文用 growing blocks 和 carriers 在线处理主体，仅对最后一个至多 $2k_N=O(\log n)$ bits 的 partial block 使用小规模 Elias encoding。最终界为 $n+\log_2n+O(\log\log n)$，而不是“Elias 编码整个消息长度后再复制内容”。

------

## 正确性检查

- $k_i$ 是 component 长度，完整 block 长度为 $2k_i$；
- block size 由已经看到的流规模决定，不要求预知最终 $n$；
- $\mathrm{eof}$ 位于 block $N-2$，被替换的 $b$ 保存在尾部；
- 3-block lag 延迟输出，没有回写已输出数据；
- Elias code 只处理最后 partial block；
- 两类逐 block loss 都由同一调和级数分析得到 $O(\log\log n)$；
- decoder 读到 output block $N-1$ 即恢复并发现 $\mathrm{eof}$；
- 总长度保留 $\log_2n$ 主项，复杂度是 per block $O(1)$，不是全程 $O(1)$。
