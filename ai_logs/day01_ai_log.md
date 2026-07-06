# Day 1 AI Log

## 今日任务背景

今天的阅读内容集中在论文 *Changing Base without Losing Space* 的 Abstract 和 Introduction 部分，目标是先理解论文的研究对象、两个核心问题、主要贡献和后续技术路线。

本日使用 AI 的目的不是让 AI 代替阅读论文，而是围绕论文内容进行辅助理解，尤其是：

- 明确论文提出的两个核心问题；
- 理解为什么向量表示问题中存在“空间最优”和“局部访问”的矛盾；
- 理解在线 prefix-free encoding 为什么重要；
- 理解 arithmetic coding 为什么不能直接解决本文的问题；
- 将 Abstract 和 Introduction 中的内容整理为第一天 notes 的基础材料。

本日 AI 输出均只作为理解辅助，写入 notes 前需要对照论文原文核验。

---

## AI 使用记录

## Prompt 1：论文核心问题与贡献梳理

**我提出的问题：**

> 请根据论文 *Changing Base without Losing Space* 的 Abstract 和 Introduction，概括这篇论文主要解决了什么问题，两个核心结果分别是什么，以及它们为什么重要。

**使用 AI 的目的：**

- 帮助快速建立论文整体框架；
- 确认 Abstract 中两个主要结果的含义；
- 为 `notes/day01_paper_overview.md` 的“内容总结”部分提供素材。

**AI 回答摘要：**

AI 将论文的主要贡献概括为两点。

第一，论文研究如何用信息论最优空间表示一个向量 `A[1..n]`，其中每个元素来自大小为 `Σ` 的字母表。理论最优空间是：

```text
⌈n log₂ Σ⌉ bits
```

传统整体编码可以达到该空间，但局部访问差；逐元素定长编码支持局部访问，但当 `Σ` 不是 2 的幂时会浪费线性空间。论文的第一个结果是：在 Word RAM 模型下，可以用 `⌈n log₂ Σ⌉` bits 表示该向量，并支持任意元素 `O(1)` 时间读写。该结果对应论文 Theorem 1。论文 Abstract 和 Introduction 中也明确表述了这一点。

第二，论文研究长度未知、在线到达的 bit stream 的 prefix-free encoding。AI 指出，传统 Elias codes 虽然接近最优，但需要提前知道输入长度，不适合低空间在线编码；朴素 block-marker 方法则会带来线性冗余。论文的第二个结果是构造长度为：

```text
n + log₂ n + O(log log n)
```

的在线 prefix-free code，且编码、解码只需要 `O(log n)` bits 空间、按 word 处理为常数时间。该结果对应论文 Theorem 2。

**采纳内容：**

采纳 AI 对论文两个核心问题的概括，并将其整理进 notes 的“内容总结”部分。

具体采纳内容包括：

- 向量表示问题的核心矛盾是“整体编码空间最优但局部性差，逐元素编码局部性好但有冗余”；
- Theorem 1 的核心是 `⌈n log₂ Σ⌉` bits 与 `O(1)` read/write 同时成立；
- Theorem 2 的核心是在线 prefix-free encoding 达到 `n + log₂ n + O(log log n)`；
- 两个结果都与“changing base without losing space”这一主题有关。

**未采纳或暂不使用内容：**

AI 对“后续相关研究”的简略提及暂未采纳，因为 Day 1 只阅读 Abstract 和 Introduction，related work 需要后续单独检索和核验。

**人工核验情况：**

已对照论文 Abstract 和 Introduction 初步核验。论文确实在 Abstract 中列出两个结果：最优空间向量表示与在线 prefix-free encoding。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## Prompt 2：向量表示问题的矛盾解释

**我提出的问题：**

> 为什么论文说用 `⌈n log₂ Σ⌉` bits 表示向量虽然信息论上最优，但局部性很差？为什么逐元素使用 `⌈log₂ Σ⌉` bits 又会浪费空间？请结合论文 Introduction 中的例子解释。

**使用 AI 的目的：**

- 辅助理解 Section 1.1 的问题动机；
- 将“信息论最优空间”和“局部访问”之间的矛盾解释清楚；
- 为 notes 中“向量表示问题”部分提供更严谨的表述。

**AI 回答摘要：**

AI 解释说，长度为 `n`、字母表大小为 `Σ` 的向量共有 `Σ^n` 种可能，因此信息论最优空间为：

```text
⌈log₂(Σ^n)⌉ = ⌈n log₂ Σ⌉ bits
```

一种达到该空间的方法是将整个向量看作 `[0, Σ^n - 1]` 中的一个大整数，再转为二进制。这种方式是全局编码。它的问题是，如果要读取某一个 `A[i]`，需要从整体数值中恢复对应位置的信息；如果要修改某个元素，也可能影响整个编码。因此它不是局部可访问的数据结构。

AI 进一步解释了逐元素编码的冗余来源。若每个元素单独使用 `⌈log₂ Σ⌉` bits，则总空间为：

```text
n⌈log₂ Σ⌉ bits
```

当 `Σ` 不是 2 的幂时，`⌈log₂ Σ⌉ > log₂ Σ`，每个元素都会浪费不足 1 bit 的空间，但这种浪费会重复 `n` 次，形成线性冗余。论文以十进制数字为例：`Σ = 10` 时，`log₂ 10 ≈ 3.322`，但定长二进制编码需要 4 bits，因此每个数字约浪费 `0.68` bits，总冗余约为 `0.68n` bits。

**采纳内容：**

采纳 AI 对两种表示方法缺陷的解释，并写入 notes 的“向量表示问题”部分。

采纳后的表述重点是：

```text
整体 base conversion：
空间最优，但局部访问差。

逐元素定长编码：
局部访问好，但当 Σ 不是 2 的幂时产生线性冗余。
```

**未采纳或暂不使用内容：**

AI 曾用“压缩整本书再修改某一页”的类比解释整体编码的局部性问题。该类比较直观，但不够正式，因此没有写入最终 notes。

**人工核验情况：**

已对照 Section 1.1。论文明确说明整体 base conversion 有 poor locality，并且指出逐元素使用 `⌈log₂ Σ⌉` bits 在 `Σ` 不是 2 的幂时会有 `Ω(n)` bits 冗余。十进制数字浪费约 `0.68n` bits 的例子也来自论文原文。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## Prompt 3：在线 Prefix-Free Encoding 的动机

**我提出的问题：**

> 请解释论文中 online prefix-free encoding 的问题背景。为什么 Elias codes 不适合这里的在线低空间场景？为什么朴素地给每个 block 加结束标记会导致线性冗余？

**使用 AI 的目的：**

- 辅助理解 Section 1.2；
- 明确在线 prefix-free encoding 与普通 universal coding 的区别；
- 为 notes 中“在线 Prefix-Free Encoding 问题”部分提供素材。

**AI 回答摘要：**

AI 解释说，prefix-free encoding 的目标是使编码结果能够自我终止，即解码器可以判断消息何时结束。对于长度未知的 bit stream，编码器不能一开始就知道最终输入长度 `n`。

Elias codes 的基本思想是先编码长度 `n`，再输出数据本身。这在离线场景中是自然的，但在在线场景中存在问题：编码器起初不知道 `n`，因此不能在开头写出长度。如果等到输入结束后再写长度，就需要缓存全部输入，不满足低空间在线处理要求。

朴素在线方案是将输入分成固定大小 block，并在每个 block 后附加一个标志位，表示是否结束。若 block 大小为 `b`，则每 `b` bits 输入大约需要额外 1 bit 标记，总冗余约为 `Θ(n/b)`，因此是线性的。论文也提到实际中可能使用如 `b = 128` 的大 block，但即使比例看起来不大，从理论上仍是线性开销。

AI 进一步指出，本文的关键视角是将 eof 看成字母表扩大问题：原本 block 来自 `[2^b]`，加入 eof 后变成大小为 `2^b + 1` 的字母表。问题因此转化为如何将 `2^b + 1` 进制的流高效转回二进制，而不是给每个 block 额外支付一个完整 bit。

**采纳内容：**

采纳 AI 对 Elias code 和朴素 block-marker 方法局限性的解释。

采纳后的核心表述是：

- Elias codes 接近最优，但不适合低空间在线编码，因为长度需要预先写入；
- 每个 block 加结束标记会导致线性冗余；
- 本文将 eof 引入后的问题视为 base conversion 问题，而不是简单标记位问题。

**未采纳或暂不使用内容：**

AI 曾进一步展开 Elias delta-code 和 omega-code 的具体编码步骤。Day 1 notes 只需要问题动机，因此没有展开写入。

**人工核验情况：**

已对照 Section 1.2。论文确实讨论了 Elias codes、朴素 block-marker 方法，以及 Maurer 和 Sjödin 关于小空间在线 prefix-free encoding 的线性冗余猜想。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## Prompt 4：Prefix-Free Encoding 的密码学意义

**我提出的问题：**

> 论文为什么强调 prefix-free encoding 在密码学中重要？请结合 cascade construction 和 extension attack 解释，但不要展开太多密码学细节。

**使用 AI 的目的：**

- 辅助理解 Introduction 中 Applications to Cryptography 部分；
- 用较精炼的方式解释 prefix-free encoding 的安全意义；
- 为 notes 中相关段落提供表述。

**AI 回答摘要：**

AI 解释说，许多密码学构造会将长消息分成 blocks，然后通过 cascade mode 迭代处理：

```text
x₀ = IV
xᵢ = f(xᵢ₋₁, mᵢ)
output xₙ
```

如果合法消息集合不是 prefix-free，可能存在一个消息 `M₁` 是另一个消息 `M₂` 的前缀，即：

```text
M₂ = M₁ || M'
```

在这种情况下，攻击者可能利用 `M₁` 的输出状态继续处理后缀 `M'`，从而得到与 `M₂` 相关的输出。这就是 extension attack 的基本直觉。

AI 指出，prefix-free encoding 的作用是排除“一个合法消息是另一个合法消息前缀”的情况，使得消息在 cascade tree 中不会处于祖先--后代关系。论文 Introduction 中也明确指出，prefix-free encodings 是防御 hash functions、MACs、PRFs 等构造中 extension attacks 的简单方式，并在 Appendix A 中展开 CBC-MAC、cascade PRF、Merkle-Damgård hash functions 等例子。

**采纳内容：**

部分采纳。

采纳了 prefix-free encoding 与 extension attack 的基本关系，并写入 notes。表述控制在背景层面，不展开具体 CBC-MAC 或 Merkle-Damgård 攻击公式。

**未采纳或暂不使用内容：**

AI 对 CBC-MAC 和 Merkle-Damgård 的具体攻击示例暂未写入 Day 1 notes，因为这属于后续深入阅读 Appendix A 时应处理的内容。

**人工核验情况：**

已对照 Introduction 中 Applications to Cryptography 部分。论文明确指出 prefix-free encoding 是防御 extension attacks 的简单 counter-measure，并列举了 CBC-MAC、cascade PRF、Merkle-Damgård hash functions 等应用。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## Prompt 5：Arithmetic Coding 为什么不够

**我提出的问题：**

> Arithmetic coding 从信息论角度可以接近最优，为什么论文仍然说它不能满足本文需求？请重点解释 worst-case locality 的问题。

**使用 AI 的目的：**

- 辅助理解 Section 1.3；
- 区分本文技术和 arithmetic coding；
- 为 notes 中“Arithmetic Coding 对比”部分提供素材。

**AI 回答摘要：**

AI 解释说，arithmetic coding 的确可以在整体压缩率上接近熵界。对于均匀分布的 `Σ` 字母表，理论长度接近：

```text
n log₂ Σ + O(1)
```

但 arithmetic coding 是一种全局或强上下文依赖的编码。其基本过程是将输入序列映射为 `[0,1]` 中的一个区间，每读一个符号就进一步细分区间。某个位置的符号会影响后续区间的缩放和输出时机。

在实际 bounded precision 实现中，编码器可能维护 outstanding bits。当当前区间暂时无法确定输出 bit 时，会延迟输出；当区间最终跨过某些边界后，可能一次性输出一串 bits。这导致一个符号对应的编码输出可能依赖周围大量符号。论文也指出，这种非局部性使得在最坏情况下无法不经线性解码就访问向量中间的符号，并且用于 prefix-free encoding 时可能出现 bursty behavior。

**采纳内容：**

采纳 AI 对 arithmetic coding 局限性的解释，并在 notes 中明确区分：

```text
Arithmetic coding：
空间效率高，但 worst-case locality 差。

本文方法：
目标是在保持局部可逆性的同时完成低冗余 base conversion。
```

**未采纳或暂不使用内容：**

AI 提到 biased distribution 下 locally-decodable arithmetic code 可能与 dictionary problem 有关。该点论文确实有所提及，但 Day 1 notes 暂不展开，以免偏离主线。

**人工核验情况：**

已对照 Section 1.3。论文明确指出 arithmetic coding 不具有本文应用所需的 worst-case locality，并说明某个字符的编码可能依赖许多周围字符，导致无法快速访问向量中间元素。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## Prompt 6：后续章节技术路线

**我提出的问题：**

> 根据 Introduction 的 Organization 部分，帮我梳理 Section 2 到 Section 5 的作用，以及 SOLE encoding、information carriers、vector representation、online prefix-free code 之间的关系。

**使用 AI 的目的：**

- 帮助理解论文整体结构；
- 为 Day 1 notes 的“后续技术路线”部分提供清晰表述；
- 确认 Section 2--5 与两个核心定理之间的对应关系。

**AI 回答摘要：**

AI 将后续章节关系概括为：

```text
SOLE Encoding
    → Information Carriers
    → Vector Representation
    → Online Prefix-Free Encoding
```

其中：

- Section 2 的 SOLE encoding 是 warm-up，展示如何在简化的在线 prefix-free 场景中将 `[B+1]` 字母表转换回 `[B]` blocks；
- Section 3 抽象出 information carriers，用于处理不能直接无冗余写入二进制内存的 spill；
- Section 4 使用 information carriers 实现最优空间向量表示，对应 Theorem 1；
- Section 5 使用 information carriers 构造在线 prefix-free code，对应 Theorem 2。

AI 还指出，SOLE 的作用不是最终理论结果本身，而是帮助读者理解“局部 base conversion”背后的机制。

**采纳内容：**

采纳该技术路线概括，并写入 notes 的最后一部分。

**未采纳或暂不使用内容：**

未展开 SOLE encoding 的具体 pass、information carrier lemma 的公式和 tree representation 的细节，因为这些属于后续阅读内容。

**人工核验情况：**

已对照 Section 1.4。论文明确说明 Section 2 介绍简单 prefix-free encoding，Section 3 形成 information carriers，Section 4 用于 vector representation，Section 5 用于 online prefix-free encoding。

**对应修改文件：**

- `notes/day01_paper_overview.md`

------

## 今日采纳情况汇总

| 咨询主题                          | 采纳情况 | 写入位置                                                  |
| --------------------------------- | -------- | --------------------------------------------------------- |
| 论文两个核心问题与贡献            | 完全采纳 | `notes/day01_paper_overview.md` 的内容总结                |
| 向量表示的空间与局部性矛盾        | 完全采纳 | `notes/day01_paper_overview.md` 的向量表示问题            |
| 在线 prefix-free encoding 动机    | 完全采纳 | `notes/day01_paper_overview.md` 的在线编码问题            |
| prefix-free encoding 的密码学意义 | 部分采纳 | 只保留 extension attack 直觉，不展开具体攻击              |
| arithmetic coding 的局部性问题    | 完全采纳 | `notes/day01_paper_overview.md` 的 arithmetic coding 对比 |
| Section 2--5 技术路线             | 完全采纳 | `notes/day01_paper_overview.md` 的后续技术路线            |

------

## 人工核验记录

本日 AI 输出中已核验的内容主要来自论文 Abstract 和 Introduction。

已核验内容包括：

- 论文第一个结果是使用 `⌈n log₂ Σ⌉` bits 表示向量并支持 `O(1)` 读写；
- 论文第二个结果是长度为 `n + log₂ n + O(log log n)` 的在线 prefix-free encoding；
- 朴素逐元素编码在 `Σ` 不是 2 的幂时会产生线性冗余；
- 十进制数字逐元素编码会浪费约 `0.68n` bits；
- Elias codes 不适合低空间在线编码；
- block-marker 方法带来线性冗余；
- prefix-free encoding 与防御 extension attack 有关；
- arithmetic coding 虽接近熵界，但 worst-case locality 不满足本文需求；
- Section 2--5 的组织结构分别对应 SOLE、information carriers、vector representation 和 online prefix-free code。

尚未深入核验内容包括：

- SOLE encoding 的具体不等式；
- information carrier lemma 的完整证明；
- Theorem 1 中精确达到 `⌈n log₂ Σ⌉` 的最后空间处理；
- Theorem 2 中 `O(log log n)` 项的详细来源；
- 论文提到的相关研究及论文发表后的后续工作。

这些尚未核验的内容不会在 Day 1 notes 中写成已证明结论。



