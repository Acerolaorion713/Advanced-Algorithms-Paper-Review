# Day 4 AI Log

## 今日任务背景

今天的阅读内容集中在论文 *Changing Base without Losing Space* 的 Section 2，目标是理解论文中的第一个具体技术构造：SOLE prefix-free encoding。

本日使用 AI 的目的不是让 AI 代替阅读论文，而是围绕论文内容进行辅助理解，尤其是：

- 理解 SOLE encoding 要解决什么问题；
- 理解为什么加入 `eof` 后，问题变成从 `[B + 1]` 到 `[B]` 的局部 base conversion；
- 梳理 Basic SOLE 的两个 pass；
- 理解 Pass 1 中 `[B + 1]^2 -> [B - 4i] × [B + 4i + 4]` 的合法性；
- 理解 Pass 2 中重新分组回到 `[B]^2` 的原因；
- 理解 SOLE 的终止处理和 prefix-free 性质；
- 理解 Property 3 所表达的局部性；
- 核验论文中 `n = 3, B = 32, M = (5, 7, 31)` 的手算例子；

本日 AI 输出均只作为理解辅助，写入 notes 或 review 前需要对照论文 Section 2 核验。

---

## AI 使用记录

## Prompt 1：SOLE Encoding 的整体目标

**我提出的问题：**

> 请根据论文 *Changing Base without Losing Space* 的 Section 2，解释 SOLE encoding 要解决什么问题，为什么它可以看作从 `[B + 1]` 到 `[B]` 的局部 base conversion，以及它和前一天阅读的 online prefix-free encoding 动机有什么关系。

**使用 AI 的目的：**

辅助理解 SOLE encoding 的问题背景和技术位置，为 `notes/day04_sole_encoding.md` 的“内容总结”和“SOLE Encoding 的问题设定”部分提供素材。

**AI 回答摘要：**

AI 解释说，SOLE encoding 是论文在 Section 2 中给出的一个 warm-up construction。Day 3 中已经说明，为了让在线流具有 prefix-free 性质，可以在普通 block 字母表 `[B]` 中加入一个特殊的 `eof` 符号。这样每个符号的字母表大小从 `B` 扩大到 `B + 1`。

如果使用朴素方法，就会给每个 block 加一个额外 marker bit，从而产生线性冗余。SOLE 的目标是避免这种 integral-bit waste：它不是显式给每个 block 加 bit，而是把来自 `[B + 1]` 的符号流重新编码成普通 `[B]` blocks。因此，SOLE 可以看作一个局部、在线的 base conversion 构造。

AI 还指出，SOLE 虽然不是论文最终的 online prefix-free code，但它展示了全文核心思想：通过局部可逆的换基技巧，在不牺牲太多空间的情况下保留局部性。

**采纳内容：**

采纳了 AI 对 SOLE 总体目标的解释，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `内容总结`
- `SOLE Encoding 的问题设定`
- `我的理解`

采纳后的核心表述是：

```text
SOLE 的目标是把加入 eof 后的 [B + 1] 字母表上的流重新编码回普通 [B] blocks；
它避免了朴素 block-marker 方法中每个 block 额外浪费 1 bit 的问题；
它是后续 information carrier lemma 的 warm-up construction。
```

**未采纳或暂不使用内容：**

AI 曾把 SOLE 和一般 arithmetic coding 的思想进行类比。该类比有一定启发性，但 Day 4 的重点是论文 Section 2 的具体构造，因此没有写入 Day 4 notes。

**人工核验情况：**

已对照论文 Section 2 和 Section 1.4。论文确实说明 Section 2 先给出一个非常简单的 prefix-free encoding 算法，并且该算法将加入 `eof` 后的字母表转换回二进制 blocks。论文 Section 1.4 也说明 SOLE 是后续 information carriers 的直觉来源。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

------

## Prompt 2：Pass 1 的区间分解

**我提出的问题：**

> 请解释 SOLE encoding 的 Pass 1。为什么要把两个来自 `[B + 1]` 的符号看成一个大数，并分解到 `[B - 4i] × [B + 4i + 4]`？为什么不等式 `(B + 1)^2 <= (B - 4i)(B + 4i + 4)` 可以保证这个转换合法？

**使用 AI 的目的：**

辅助理解 Pass 1 的具体算法和正确性，为 notes 中“Pass 1：Short-Odd Long-Even 转换”部分提供素材。

**AI 回答摘要：**

AI 解释说，Pass 1 处理位置 `(2i + 1, 2i + 2)` 上的一对符号。由于每个符号来自 `[B + 1]`，所以一对符号共有 `(B + 1)^2` 种可能，可以看成一个大数 `z`。

Pass 1 的目标是把这个大数重新表示成两个值：一个属于较短区间 `[B - 4i]`，另一个属于较长区间 `[B + 4i + 4]`。具体做法是对 `B - 4i` 做除法：

```text
z = quotient · (B - 4i) + remainder
```

其中：

```text
remainder ∈ [B - 4i]
quotient  ∈ [B + 4i + 4]
```

只要目标区间的容量至少等于原始输入数量，即：

```text
(B + 1)^2 <= (B - 4i)(B + 4i + 4)
```

这种商和余数的表示就是合法的，并且可以唯一恢复原来的 `z`，从而恢复原来的两个输入符号。

**采纳内容：**

采纳了 AI 对 Pass 1 的解释，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `Pass 1：Short-Odd Long-Even 转换`

采纳后的重点包括：

```text
Pass 1 处理 pair (2i + 1, 2i + 2)；
两个 [B + 1] 符号被合成一个大数；
通过除以 B - 4i 得到商和余数；
余数落入 [B - 4i]；
商落入 [B + 4i + 4]；
容量不等式保证映射合法且可逆。
```

**未采纳或暂不使用内容：**

AI 曾进一步尝试把不等式完全展开并推导到 `B >= 2(2i + 1)^2 - 3/2`。该推导与论文一致，但 Day 4 notes 为了保持精炼，只保留了核心容量不等式，没有完整展开每一步代数推导。

**人工核验情况：**

已对照论文 Section 2.1。论文确实将 Pass 1 描述为把位置 `(2i + 1, 2i + 2)` 上的两个元素从 `[B + 1]^2` 分解到 `[B - 4i] × [B + 4i + 4]`，并给出关键条件：

```text
(B + 1)^2 <= (B - 4i)(B + 4i + 4)
```

论文也说明该分解通过除以 `B - 4i` 完成。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

------

## Prompt 3：Pass 2 和重新分组

**我提出的问题：**

> 请解释 SOLE encoding 的 Pass 2。为什么 Pass 1 之后要错位重新分组？为什么 `[B + 4i] × [B - 4i]` 可以重新编码成 `[B]^2`？

**使用 AI 的目的：**

辅助理解 SOLE 的第二阶段，尤其是 short range 和 long range 为什么能在重新分组后抵消，为 notes 中“Pass 2：重新分组回到 `[B]^2`”部分提供素材。

**AI 回答摘要：**

AI 解释说，Pass 1 之后，区间大小会呈现交替结构：奇数位置略短，偶数位置略长。Pass 2 不再按照 Pass 1 的原始 pair 分组，而是错位分组，将一个 long range 和下一个 short range 配对。

典型的一对是：

```text
[B + 4i] × [B - 4i]
```

这对区间的总容量为：

```text
(B + 4i)(B - 4i) = B^2 - 16i^2 < B^2
```

因此，这一对的所有可能值可以被注入到 `[B]^2` 中。由于 `[B]^2` 正好对应两个普通的 `b`-bit blocks，Pass 2 就把交替的非标准区间重新转换回普通 blocks。

AI 强调，SOLE 的巧妙之处在于 Pass 1 不是直接把每一对变成 `[B]^2`，而是先制造 short / long 交替结构，再通过 Pass 2 的错位分组让它们合并回标准区间。

**采纳内容：**

采纳了 AI 对 Pass 2 的解释，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `Pass 2：重新分组回到 [B]^2`
- `我的理解`

采纳后的核心表述是：

```text
Pass 1 后形成交替的 short / long ranges；
Pass 2 错位重新分组；
[B + 4i] × [B - 4i] 的容量小于 B²；
因此可以编码成两个普通 [B] blocks。
```

**未采纳或暂不使用内容：**

AI 曾使用“借位”和“抵消”类比来解释 long/short 结构。这个类比直观但不够正式，因此只保留了“long 和 short 在容量上互补”的表达，没有使用过于口语化的说法。

**人工核验情况：**

已对照论文 Section 2.1。论文确实说明 Pass 2 考虑位置 `(2i, 2i + 1)`，其范围分别为 `[B + 4i]` 和 `[B - 4i]`，并利用：

```text
(B + 4i)(B - 4i) < B²
```

将其重新组合为 `[B]^2` 中的 double-block。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

------

## Prompt 4：终止处理和 Prefix-Freeness

**我提出的问题：**

> 请解释 Basic SOLE 的终止处理。为什么在 `n` 为奇数时，`eof` 会出现在偶数位置？为什么算法还要在 Pass 1 后人工加入一个 `0`？解码器为什么能在读过编码结尾之前发现 `eof` 并停止？

**使用 AI 的目的：**

辅助理解 SOLE 的 termination behavior 和 prefix-free 性质，为 notes 中“终止处理”部分提供素材。

**AI 回答摘要：**

AI 解释说，Basic SOLE 假设输入 block 数 `n` 是奇数，因此加入 `eof` 后，`eof` 位于位置：

```text
n + 1 = 2i
```

这是一个偶数位置。Pass 1 按照 pair 处理输入，所以 `eof` 会和前一个输入 block 组成一对。

Pass 1 结束后，最后会留下一个处在 long range 中的值。为了让 Pass 2 能够完成最后一次重新分组，算法人工插入一个 `0`，并把它看作属于：

```text
[B - 4i - 4]
```

的范围。这样最后一对也可以通过 Pass 2 转换成普通 `[B]` blocks。

解码时，一旦恢复出 `eof`，解码器立即停止。由于 Property 3 保证恢复包含 `eof` 的输入 blocks 只需要读取附近常数个输出 blocks，所以解码器能够在读过编码结尾之前看到 `eof`，不会越界读取。这样编码具有 prefix-free 性质。

**采纳内容：**

采纳了 AI 对终止处理的解释，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `终止处理`

采纳后的重点包括：

```text
Basic SOLE 假设 n 为奇数；
eof 出现在偶数位置 n + 1 = 2i；
算法人工加入一个 0 来补齐最后一组；
decoder 恢复出 eof 后立即停止；
Property 3 保证 decoder 不需要读过文件末尾。
```

**未采纳或暂不使用内容：**

AI 曾展开讨论 `n` 为偶数时如何通过 padding 变成奇数情形。该内容在论文中有提及，但 Day 4 notes 保持精炼，只记录 Basic SOLE 的主要终止逻辑。

**人工核验情况：**

已对照论文 Section 2.1 的 Termination 部分。论文确实说明 Basic SOLE 假设 `n` 为奇数，即 `eof` 出现在偶数位置 `n + 1 = 2i`，并在 Pass 1 后人工插入一个 `0`，范围为 `[B - 4i - 4]`。论文也明确指出解码算法在恢复 `eof` 后立即停止，并用 Property 3 说明不会读过编码末尾。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

------

## Prompt 5：局部性性质 Property 3

**我提出的问题：**

> 请解释论文 Section 2 中 Property 3 的含义。为什么说 SOLE 的编码和解码具有局部性？这个性质为什么对后续向量表示和 information carrier 技术重要？

**使用 AI 的目的：**

辅助理解 SOLE 的局部性意义，为 notes 中“局部性性质”和“我的理解”部分提供素材。

**AI 回答摘要：**

AI 解释说，Property 3 说明 SOLE 的输出不是全局依赖输入的。具体来说，输出 blocks `2i` 和 `2i + 1` 只需要输入 blocks `2i - 1, ..., 2i + 2` 就能计算出来；反过来，输入 blocks `2i + 1` 和 `2i + 2` 也只需要输出 blocks `2i, ..., 2i + 3` 就能恢复。

这意味着编码和解码都是局部的：修改一个输入 block 不会影响整个编码，只需要重算附近常数个输出 blocks。AI 指出，这一点和普通整体 base conversion 或 arithmetic coding 不同，后两者通常没有 worst-case locality。

AI 进一步说明，SOLE 的局部性是后续 information carrier lemma 和 vector representation 的思想来源。后文要实现的是在最优空间下支持 `O(1)` read/write，而这种目标必须依赖局部可逆的表示方式。

**采纳内容：**

采纳了 AI 对 Property 3 的解释，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `局部性性质`
- `我的理解`
- `可以写入 review 的段落草稿`

采纳后的核心表述是：

```text
SOLE 的编码和解码只依赖常数个相邻 blocks；
修改局部输入只影响局部输出；
这种局部可逆性是后续 O(1) read/write 的基础直觉。
```

**未采纳或暂不使用内容：**

AI 曾把 Property 3 解释成“类似滑动窗口编码”。该比喻可以帮助理解，但正式 notes 中没有使用，以避免和论文术语混淆。

**人工核验情况：**

已对照论文 Section 2.1。论文确实给出 Property 3，并明确说明输出 blocks `2i` 和 `2i + 1` 可由输入 blocks `2i - 1, ..., 2i + 2` 计算，输入 blocks `2i + 1` 和 `2i + 2` 可由输出 blocks `2i, ..., 2i + 3` 解码。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

------

## Prompt 6：论文中的 `n = 3, B = 32` 手算例子

**我提出的问题：**

> 请根据论文 Section 2.2，手算 `n = 3, B = 32, M = (5, 7, 31)` 的 SOLE encoding 过程。重点解释 Pass 1 和 Pass 2 分别得到什么，为什么最终输出是 `(12, 3, 26, 6, 1)`。

**使用 AI 的目的：**

辅助核验论文中的具体例子，确保 notes 中的手算过程正确。

**AI 回答摘要：**

AI 按论文例子重新计算了 SOLE encoding 的过程。

首先，`B = 32`，普通 block 来自 `[32]`。加入 `eof` 后，字母表变成 `[33]`，其中 `eof = 32`。输入：

```text
(5, 7, 31)
```

扩展为：

```text
(5, 7, 31, 32, 0)
```

第一对 `(5, 7)`：

```text
7 · 33 + 5 = 236
236 = 7 · 32 + 12
```

所以得到：

```text
(12, 7)
```

第二对 `(31, 32)`：

```text
32 · 33 + 31 = 1087
1087 = 38 · 28 + 23
```

所以得到：

```text
(23, 38)
```

Pass 1 后得到：

```text
(12, 7, 23, 38, 0)
```

Pass 2 重新分组为：

```text
12
(7, 23)
(38, 0)
```

其中：

```text
23 · 36 + 7 = 835 = 26 · 32 + 3
```

得到：

```text
(3, 26)
```

最后：

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

**采纳内容：**

采纳了 AI 对手算例子的整理，并写入 `notes/day04_sole_encoding.md` 的以下部分：

- `论文中的小例子`

采纳后的重点是完整保留关键算式，方便后续写 review 时选择是否压缩。

**未采纳或暂不使用内容：**

AI 曾继续手算 decoding 过程。论文 Section 2.2.2 中确实包含 decoding，但 Day 4 notes 为了精炼，暂时只完整记录 encoding 方向。Decoding 部分后续如果 review 需要更详细解释正确性，可以再补充。

**人工核验情况：**

已对照论文 Section 2.2。论文中的例子确实设置 `n = 3`、`B = 32`、输入 `(5, 7, 31)`，并最终得到输出：

```text
(12, 3, 26, 6, 1)
```

中间计算与论文表格和文字说明一致。

**对应修改文件：**

- `notes/day04_sole_encoding.md`
- 后续可用于 `review/sections/05_sole_encoding.tex`

