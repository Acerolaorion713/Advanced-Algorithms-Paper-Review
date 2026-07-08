# Day 3 AI Log

## 今日任务背景

今天的阅读内容集中在论文 *Changing Base without Losing Space* 的 Section 1.2 和 Appendix A，目标是理解论文第二个核心结果的背景：online prefix-free encoding 为什么重要，为什么传统 Elias codes 不适合本文的在线低空间场景，为什么朴素 block-marker 方法会产生线性冗余，以及 prefix-free encoding 在密码学 cascade 构造中为什么能够防止 extension attack。

本日使用 AI 的目的不是让 AI 代替阅读论文，而是围绕论文内容进行辅助理解，尤其是：

- 理解 prefix-free encoding / universal coding / self-delimiting code 的关系；
- 理解 Elias delta-code 和 Elias omega-code 的基本作用；
- 梳理 Elias codes 在 online low-memory setting 中的局限；
- 解释朴素 block-marker 方法的线性冗余来源；
- 理解 EOF 符号如何把问题转化为从 `[2^b + 1]` 到 `[2^b]` 的 base conversion；
- 理解 Maurer 和 Sjödin 关于小空间 online prefix-free encoding 的线性冗余猜想；
- 理解 CBC-MAC、cascade PRF、Merkle-Damgård hash functions 中 extension attack 的直觉；

---

## AI 使用记录

## Prompt 1：Online Prefix-Free Encoding 问题背景

**我提出的问题：**

> 请根据论文 *Changing Base without Losing Space* 的 Section 1.2，解释 online prefix-free encoding 要解决什么问题，为什么这个问题和普通 universal coding 不完全一样，以及为什么本文强调低空间和在线处理。

**使用 AI 的目的：**

辅助理解 Section 1.2 的问题定义和研究动机，为 `notes/day03_prefix_free_encoding.md` 的开头部分提供素材。

**AI 回答摘要：**

AI 解释说，prefix-free encoding 的目标是让编码结果具有自终止性，即解码器能够仅根据编码内容判断消息何时结束，而不需要额外获得消息长度。信息论中常把这类问题称为 universal coding，计算机科学和密码学中也常称为 self-delimiting code 或 prefix-free code。

AI 进一步指出，本文关注的不是一般离线 universal coding，而是 online setting：输入 bit stream 逐步到达，最终长度 `n` 事先未知。编码器不能等到读完整个输入后再统一编码，因为这样需要缓存全部输入，不满足论文追求的 `O(log n)` bits memory。因此，本文要求编码和解码都能一遍完成，并且只维护很小的状态。

**采纳内容：**

采纳了 AI 对 online prefix-free encoding 问题的基本概括，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `Prefix-Free Encoding 问题`
- `内容总结`

采纳后的核心表述是：

```text
prefix-free encoding 需要让消息编码隐式包含终止信息；
本文的困难在于输入长度 n 在线未知；
编码器不能缓存整个输入；
目标是同时满足 prefix-free、online、low-memory 和 low-overhead。
```

**未采纳或暂不使用内容：**

AI 曾扩展解释一般前缀码与 Huffman coding 的关系。该内容虽然与 prefix-free code 有关，但不是本文 Section 1.2 的重点，因此没有写入 Day 3 notes。

**人工核验情况：**

已对照论文 Section 1.2。论文确实说明，当要表示长度不固定的 `n` bits 信息时，需要使用 prefix-free code；在 information theory 中这类任务称为 universal coding，self-delimiting code 也是常见术语。论文也明确指出，在许多自然应用中消息在线到达，长度事先未知，因此 Elias codes 不适合该设置。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`

------

## Prompt 2：Elias Codes 的作用与局限

**我提出的问题：**

> 请解释 Elias delta-code 和 Elias omega-code 在论文 Section 1.2 中起什么作用。为什么它们可以作为离线 universal coding 的 textbook solution，但不能直接用于本文的 online low-memory setting？

**使用 AI 的目的：**

辅助理解论文为什么先介绍 Elias codes，以及 Elias codes 和本文 online prefix-free encoding 的差别。

**AI 回答摘要：**

AI 解释说，Elias codes 的基本思想是先编码长度 `n`，再输出原始数据。Elias delta-code 可以用 `n + O(log n)` bits 编码一个长度为 `n` 的 bit string；Elias omega-code 通过递归编码长度信息，可以达到：

```text
n + log₂ n + log₂ log₂ n + ... + O(log* n)
```

AI 指出，这些 codes 在离线场景中很好，因为编码器已经知道完整输入长度 `n`，可以先写长度描述，再写数据。但在在线场景中，编码器一开始并不知道最终 `n`。如果为了使用 Elias code 而等到最后再写长度，就必须缓存整个输入，空间复杂度会变成 `Ω(n)`，不符合本文的 `O(log n)` bits memory 目标。

**采纳内容：**

采纳 AI 对 Elias codes 的总结，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `Elias Codes 的作用与局限`
- `关键公式整理`

采纳后的重点包括：

```text
Elias delta-code: n + O(log n)
Elias omega-code: n + log₂ n + log₂ log₂ n + ... + O(log* n)
Elias codes 的局限：长度信息需要先于数据出现，不适合在线未知长度输入。
```

**未采纳或暂不使用内容：**

AI 曾尝试给出 Elias delta-code 的完整 bit-level 编码例子。该例子对理解 Elias code 有帮助，但 Day 3 notes 的重点是问题动机，不是详细讲解 Elias code 算法，因此没有展开。

**人工核验情况：**

已对照论文 Section 1.2。论文确实使用 Elias delta-code 和 Elias omega-code 作为 textbook solution，并指出 Elias omega-code 的长度达到 `n + log₂ n + log₂ log₂ n + ... + O(log* n)`，同时说明 Elias codes 在本文的在线场景中不可用。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`

------

## Prompt 3：朴素 Block-Marker 方法和线性冗余

**我提出的问题：**

> 为什么论文说给每个 block 加一个结束标记会导致线性冗余？请结合 block size `b`、输入长度 `n` 和 overhead `Θ(n/b)` 解释。

**使用 AI 的目的：**

辅助理解朴素 online prefix-free encoding 方法的代价，并为 notes 中“朴素 Block-Marker 方法”部分提供素材。

**AI 回答摘要：**

AI 解释说，朴素方法会把输入流切成固定大小的 `b`-bit blocks，并在每个 block 后加一个标记 bit，用来说明当前 block 是否为最后一个 block。这样编码器可以在线工作，因为每读到一个 block 就能立即输出数据和标记。

但该方法每处理 `b` bits 数据就额外付出 `1` bit，因此对于总长度为 `n` 的输入，额外 bit 数约为：

```text
Θ(n / b)
```

如果 `b` 是固定常数，例如实际系统中常见的 `b = 128`，则 `Θ(n / b)` 仍然是关于 `n` 的线性冗余。论文指出，`b = 128` 时 overhead 接近 1%，工程上可能可接受，但理论上仍然是一种为了编码长度而付出的线性成本。

**采纳内容：**

采纳 AI 对 block-marker 方法的冗余分析，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `朴素 Block-Marker 方法`
- `关键公式整理`

采纳后的核心表述是：

```text
每个 b-bit block 额外付出 1 bit；
总冗余为 Θ(n/b)；
当 b 固定时，这是线性冗余；
该方法体现了 memory 和 redundancy 之间的朴素线性 trade-off。
```

**未采纳或暂不使用内容：**

AI 曾补充讨论实际网络协议中的分帧和长度字段设计。该内容与工程背景有关，但不是论文论证主线，因此没有写入 notes。

**人工核验情况：**

已对照论文 Section 1.2。论文确实提到实践中可使用 `b = 128` 的大 blocks，并在每个 block 后附加一位表示是否到达文件末尾；论文也指出这种方案理论上给出了 redundancy 与 algorithm space 之间的线性 trade-off。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`

------

## Prompt 4：EOF 与 Base Conversion 视角

**我提出的问题：**

> 请解释论文中“不要从 integral bits 的角度理解 EOF”的意思。为什么加入 EOF 后，问题可以被看作从 `[2^b + 1]` 到 `[2^b]` 的 base conversion？这个观察为什么是后续 SOLE encoding 的基础？

**使用 AI 的目的：**

辅助理解 Section 1.2 中最关键的技术转化：从结束标记问题转向局部换基问题。

**AI 回答摘要：**

AI 解释说，朴素 block-marker 方法的问题在于它把 EOF 信息理解成“每个 block 额外加一个完整 bit”。但实际上，如果每个普通 block 有 `B = 2^b` 种可能取值，那么加入一个特殊的 EOF 符号后，字母表大小只是从 `B` 增加到 `B + 1`。单个符号的信息量增加为：

```text
log₂(B + 1) - log₂ B = log₂(1 + 1/B)
```

当 `B` 很大时，这个增量远小于 `1` bit。因此真正的挑战不是给每个 block 加 bit，而是如何把来自 `[B + 1]` 字母表的流高效、局部地重新编码回 `[B]` 字母表上的 blocks。

AI 进一步说明，Section 2 的 SOLE encoding 正是从这个简化问题开始：输入 blocks 原本来自 `[B]`，加入 EOF 后来自 `[B + 1]`，算法要把它们转换成普通 `[B]` blocks，同时避免每个 block 浪费一个完整 bit。

**采纳内容：**

采纳 AI 对 EOF 与 base conversion 关系的解释，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `EOF 符号与 Base Conversion 视角`
- `冗余估算的核心直觉`
- `与 Day 4 SOLE Encoding 的关系`

采纳后的重点包括：

```text
B = 2^b；
加入 EOF 后，字母表大小从 B 变成 B + 1；
EOF 的真实信息成本是 log₂(1 + 1/B)，不是 1 bit；
问题转化为将 [B + 1] 字母表的流局部转换回 [B] blocks；
SOLE encoding 是这个思想的第一个具体构造。
```

**未采纳或暂不使用内容：**

AI 曾提前展开 SOLE encoding 的两个 pass 和 `[B - 4i] × [B + 4i + 4]` 分解。这部分属于 Day 4 阅读任务，因此 Day 3 notes 中只保留“为什么引出 SOLE”，没有详细写 SOLE 算法。

**人工核验情况：**

已对照论文 Section 1.2 和 Section 1.4。论文确实指出，关键误区是从 integral bits 的角度理解问题；加入 EOF 后可以把 `[2^b]` 扩展为 `[2^b + 1]`，如果能高效转换回二进制，就能避免每个 block 浪费一个 bit。论文 Section 1.4 也说明 Section 2 会先给出一个简单的 SOLE prefix-free encoding。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`

------

## Prompt 5：Theorem 2 的意义

**我提出的问题：**

> 请解释论文 Theorem 2 的 statement 和意义。为什么 `n + log₂ n + O(log log n)`、`O(log n)` bits memory、`O(1)` time per operation 这三个条件放在一起很强？

**使用 AI 的目的：**

辅助理解论文第二个核心结果的理论意义，并将其写入 Day 3 notes 的总结部分。

**AI 回答摘要：**

AI 解释说，Theorem 2 给出一种 universal code，可以把 `n` bits 编码为：

```text
n + log₂ n + O(log log n)
```

并且编码、解码算法只使用：

```text
O(log n) bits
```

空间，同时每次操作为常数时间。

AI 指出，该结果同时接近 Elias codes 的最优长度，又满足 Elias codes 不满足的 online low-memory 要求。与朴素 block-marker 方法相比，它避免了 `Θ(n/b)` 线性冗余。与普通离线 universal coding 相比，它不要求预先知道输入总长度。因此，这个定理反驳了“小空间 online prefix-free encoding 可能必须付出线性冗余”的直觉。

**采纳内容：**

采纳 AI 对 Theorem 2 的总结，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `Theorem 2 的意义`
- `今日阅读结论`

采纳后的核心表述是：

```text
Theorem 2 同时实现 near-optimal overhead、online processing 和 low memory；
它比 Elias codes 更适合在线场景；
它比 block-marker 方法空间效率更高；
它反驳了小空间 online PFE 必须线性冗余的直觉。
```

**未采纳或暂不使用内容：**

AI 曾尝试提前概述 Section 5 中 slowly growing blocks 和 information carriers 的完整实现。该内容属于 Day 7 的重点，因此 Day 3 notes 中只保留定理意义，不展开完整证明。

**人工核验情况：**

已对照论文 Abstract 和 Section 1.2。论文确实在 Abstract 和 Theorem 2 中说明存在一种 universal code，将 `n` bits 映射到 `n + log₂ n + O(log log n)` bits，并且编码、解码使用 `O(log n)` bits memory，按 word / operation 常数时间处理。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`

------

## Prompt 6：密码学中的 Extension Attack

**我提出的问题：**

> 请根据论文 Appendix A，解释为什么 prefix-free encoding 对 CBC-MAC、cascade PRF 和 Merkle-Damgård hash functions 很重要。什么是 extension attack？为什么 prefix-free encoding 可以避免这类问题？

**使用 AI 的目的：**

辅助理解 Appendix A 的密码学动机，并为 Day 3 notes 中“密码学中的 Prefix-Free Encoding 动机”部分提供素材。

**AI 回答摘要：**

AI 解释说，很多密码学构造都可以抽象为 cascade mode。给定压缩函数：

```text
f : {0,1}^s × {0,1}^b → {0,1}^s
```

以及初始状态 `IV`，消息 `M = m₁m₂...mₙ` 被分成 blocks 后逐步计算：

```text
x₀ = IV
xᵢ = f(xᵢ₋₁, mᵢ)
output xₙ
```

如果一个合法消息是另一个合法消息的前缀，例如：

```text
M₂ = M₁M'
```

那么：

```text
Cascade(M₂, f, IV)
= Cascade(M', f, Cascade(M₁, f, IV))
```

这意味着短消息的输出可以被用作长消息继续计算的中间状态。攻击者可能利用这种结构从短消息输出推导出长消息相关输出，这就是 extension attack。

Prefix-free encoding 避免了合法消息之间的前缀关系，使一个合法消息不可能成为另一个合法消息的前缀。因此，在 cascade 计算树中，不同合法消息不会形成祖先和后代关系，extension attack 的基础被破坏。

**采纳内容：**

采纳 AI 对 cascade mode 和 extension attack 的解释，并写入 `notes/day03_prefix_free_encoding.md` 的以下部分：

- `密码学中的 Prefix-Free Encoding 动机`
- `CBC-MAC 中的 Extension Attack`
- `Cascade PRF 中的 Extension Attack`
- `Merkle-Damgård Hash Functions 中的 Extension Attack`

采纳后的重点包括：

```text
许多密码学构造都可看作 cascade mode；
非 prefix-free 编码允许一个消息成为另一个消息的前缀；
前缀关系会导致 ancestor-descendant relation；
extension attack 利用短消息输出继续计算长消息输出；
prefix-free encoding 是防止这类攻击的自然方式。
```

**未采纳或暂不使用内容：**

AI 曾补充讨论现代 hash padding 标准和 HMAC 的工程细节。由于 Day 3 notes 主要依据论文 Appendix A，不做外部扩展，因此没有写入。

**人工核验情况：**

已对照论文 Appendix A。论文确实说明 CBC-MAC、cascade PRF、Merkle-Damgård hash functions、MAC domain extension 等都可以从 cascade mode 的角度理解，并且 prefix-free encoding 是防止 extension attack 的简单、统一方式。论文还给出了 CBC-MAC 和 cascade PRF 中的具体 extension attack 形式。

**对应修改文件：**

- `notes/day03_prefix_free_encoding.md`