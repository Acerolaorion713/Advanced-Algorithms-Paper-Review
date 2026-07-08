# Day 3 Notes: Prefix-Free Encoding and Cryptographic Motivation

## 阅读范围

本次阅读范围主要是论文 *Changing Base without Losing Space* 的 Section 1.2 和 Appendix A，重点围绕 online prefix-free encoding 的问题背景、Elias codes 的局限、朴素 block-marker 方法的线性冗余、EOF 符号带来的字母表扩大问题，以及 prefix-free encoding 在密码学构造中的作用展开。

具体来说，本次阅读关注以下主题：

prefix-free encoding / universal coding / self-delimiting code 的基本含义；

为什么长度未知的在线 bit stream 需要 prefix-free encoding；

Elias delta-code 和 Elias omega-code 的基本思想；

为什么 Elias codes 不适合本文的低空间在线场景；

朴素 block-marker 方法为什么会产生线性冗余；

为什么加入 EOF 后，问题可以被看成从 `[2^b]` 到 `[2^b + 1]` 的字母表扩大问题；

Maurer 和 Sjödin 关于小空间 online prefix-free encoding 的线性冗余猜想；

本文 Theorem 2 的意义；

prefix-free encoding 在 CBC-MAC、cascade PRF、Merkle-Damgård hash functions 和 MAC domain extension 中的作用；

extension attack 为什么说明 prefix-free 性质非常重要。

---

## 内容总结

这一天的阅读重点是理解论文第二个核心问题：如何对长度未知、在线到达的 bit stream 构造低冗余的 prefix-free encoding。Prefix-free encoding 的目标是让编码结果具有自终止性，即解码器不需要额外知道输入长度，也能判断消息在哪里结束。对于离线场景，Elias codes 是经典做法：先编码长度 `n`，再输出数据本身，可以达到 `n + O(log n)` 或更接近最优的长度。但是在本文关心的在线低空间场景中，编码器一开始并不知道最终长度 `n`，因此无法预先写出长度字段；如果等到输入结束后再写长度，就必须缓存整个输入，这与 `O(log n)` bits memory 的目标冲突。

朴素的在线方法是把输入分成固定大小的 blocks，并在每个 block 后附加一个结束标记，表示是否到达文件末尾。这个方法虽然简单且在线，但每个 block 都额外消耗一个 bit。当 block 大小 `b` 固定时，总冗余为 `Θ(n / b)`，对输入长度 `n` 来说仍然是线性的。论文指出，真正的问题不应该被理解为“每个 block 是否额外加一个 bit”，而应该被理解为 base conversion 问题：原本一个 `b`-bit block 来自大小为 `B = 2^b` 的字母表 `[B]`，加入 EOF 后，字母表大小变成 `B + 1`。如果能高效地把 `[B + 1]` 上的流重新编码成 `[B]` 上的 blocks，就可以避免每个 block 浪费一个完整 bit。

Appendix A 进一步解释了 prefix-free encoding 的密码学意义。许多 hash、MAC 和 PRF 构造都采用 cascade mode：把消息分成 blocks，然后反复调用压缩函数更新内部状态。如果消息编码不是 prefix-free，那么一个合法消息可能成为另一个合法消息的前缀，攻击者就可能利用短消息的输出继续处理额外 blocks，从而得到长消息的相关输出，这就是 extension attack。Prefix-free encoding 的作用是阻止这种前缀关系，使不同消息在 cascade 计算树中不会形成祖先和后代关系。因此，本文的 online PFE 不只是一个编码问题，也能为密码学中的 domain extension 提供更简单、更统一的处理方式。

------

## 论文具体解析

## Prefix-Free Encoding 问题

论文考虑的第二个问题是：

```text
给定一个在线到达的 bit stream，总长度为 n，但 n 事先未知。
目标是输出一个 prefix-free encoding，使编码结果能够隐式包含长度信息。
```

这里的 prefix-free 表示：任意一个合法编码都不能是另一个合法编码的前缀。这样解码器读取编码流时，一旦读到某个完整合法编码，就可以确定消息已经结束，而不会担心后面还可能接着形成另一个合法消息。

在信息论中，这类问题通常叫 universal coding。在计算机科学和密码学语境中，常见说法包括：

```text
prefix-free code
self-delimiting code
universal code
```

这些术语强调的角度略有不同，但在本文语境中，它们都指向同一个核心需求：对长度未知的数据进行自描述编码，使长度信息不需要由外部单独给出。

本文的目标不是普通离线压缩，而是更强的在线要求：

```text
输入逐步到达；
编码器不能提前知道 n；
编码器不能缓存整个输入；
编码和解码都应只使用 O(log n) bits memory；
按 word 处理时应达到常数时间。
```

因此，这个问题同时包含三个约束：

```text
prefix-free 性质；
online 处理；
low-memory 和 low-overhead。
```

---

## Elias Codes 的作用与局限

论文首先提到 Elias codes 是 universal coding 的 textbook solution。其基本思想是：把长度 `n` 编码到数据前面，然后再写出原始数据。

以 Elias delta-code 为例，编码一个长度为 `n` 的 bit vector 时，可以分三步：

```text
1. 写出若干个 0，数量与 log₂ n 的二进制长度有关，然后写一个 1；
2. 写出 n 的二进制表示；
3. 写出原始的 n bits 数据。
```

这个方法可以达到：

```text
n + O(log n) bits
```

进一步递归编码长度信息，可以得到 Elias omega-code，其长度为：

```text
n + log₂ n + log₂ log₂ n + ... + O(log* n)
```

论文指出，这个 bound 在 universal coding 中是最优量级的。

但是 Elias codes 不适合本文的在线低空间场景。原因是 Elias codes 的结构要求长度信息出现在数据之前，而在线编码器在一开始并不知道最终输入长度 `n`。如果编码器等待整个输入结束后再回头写长度，则需要保存整个输入，空间复杂度变成 `Ω(n)`，无法满足本文想要的 `O(log n)` bits memory。

因此，Elias codes 说明了“理论上可以用接近最优的长度编码未知长度数据”，但没有解决“如何在线、低空间地做到这一点”。

---

## 朴素 Block-Marker 方法

一种自然的在线方法是把输入流切成固定大小的 blocks。设每个 block 有 `b` bits，则每个普通 block 来自：

```text
[2^b]
```

如果需要判断文件是否结束，可以在每个 block 后额外加一个标记 bit：

```text
0 表示后面还有 block；
1 表示当前 block 是最后一个 block。
```

这种方法的优点是简单、在线、易解码。编码器每读一个 block 就可以立即输出这个 block 和对应的标记位，不需要知道未来输入。

但问题是它会产生线性冗余。每 `b` bits 输入就额外付出 `1` bit，因此对于总长度为 `n` 的输入，总 overhead 大约是：

```text
Θ(n / b)
```

当 `b` 是固定常数时，例如实际系统中常见的 `b = 128`，这个 overhead 对 `n` 来说仍然是线性的。论文指出，`b = 128` 时开销接近 1%，在工程上也许可以接受，但理论上这是为了表示长度而付出的相当高代价。

这类方法本质上形成了一个线性 trade-off：

```text
buffer 越大，冗余比例越小；
buffer 越小，冗余越大；
如果 memory 很小，就很难避免线性冗余。
```

Maurer 和 Sjödin 曾猜想，小空间 online prefix-free encoding 也许必须付出线性冗余。本文的结果正是要反驳这种直觉。

---

## EOF 符号与 Base Conversion 视角

论文最关键的观察是：朴素 block-marker 方法的问题在于它把 EOF 信息当成一个完整 bit 来处理，而不是把它看成字母表大小的轻微增加。

设：

```text
b = block size
B = 2^b
[B] = {0, 1, ..., B - 1}
```

原始输入中，每个 block 是一个 `[B]` 中的元素。为了让流变成 prefix-free，可以在普通字母表 `[B]` 之外加入一个特殊符号：

```text
eof
```

于是字母表大小从：

```text
B
```

变成：

```text
B + 1
```

这一步非常重要。加入 EOF 并不意味着每个 block 都必须额外使用一个完整 bit，而只是把每个符号的可能取值数量从 `B` 增加到 `B + 1`。信息量的增加是：

```text
log₂(B + 1) - log₂ B
= log₂(1 + 1/B)
```

当 `B` 很大时，这个增量远小于 `1` bit。

因此，真正的问题可以重写为：

```text
如何把一个来自 [B + 1] 字母表的流，高效、局部、在线地转换回 [B] 字母表上的 blocks？
```

这就是本文题目 “Changing Base without Losing Space” 在 prefix-free encoding 场景下的含义。EOF 不是简单的附加标记位，而是引出了一个局部换基问题。

如果能够把 `[B + 1]` 上的流高效转换为 `[B]` 上的流，就可以避免每个 block 浪费一个完整 bit，从而突破朴素方法的线性冗余。

---

## 冗余估算的核心直觉

论文给出一个简化估算来说明为什么这种思路有希望。

假设输入由 `n` 个 `b`-bit blocks 组成，并且暂时保证：

```text
n ≤ 2^b
```

加入 EOF 后，需要编码的是 `n + 1` 个来自大小为 `2^b + 1` 的字母表的符号。若能够理想地进行 base conversion，则所需 bit 数约为：

```text
(n + 1) log₂(2^b + 1)
```

原始数据本身有：

```text
n · b
```

bits。因此 overhead 为：

```text
(n + 1) log₂(2^b + 1) - n · b
```

将 `2^b + 1` 写成 `2^b(1 + 1/2^b)`，可得：

```text
log₂(2^b + 1)
= b + log₂(1 + 1/2^b)
```

所以 overhead 近似为：

```text
b + n log₂(1 + 1/2^b)
```

由于：

```text
log₂(1 + 1/2^b) 很小
```

并且有 `n ≤ 2^b`，因此额外开销可以控制在：

```text
b + O(1)
```

这说明，只要能避免“每个 block 多付 1 bit”的整数浪费，EOF 的真实信息成本其实很低。

为了去掉 `n ≤ 2^b` 的假设，论文后续使用 slowly growing blocks，即随着已经处理的数据规模增加，逐步增大 block size。这样可以把总 overhead 控制到：

```text
O(log n)
```

更完整的 Section 5 结果进一步达到：

```text
n + log₂ n + O(log log n)
```

---

## Theorem 2 的意义

论文 Theorem 2 表述为：存在一种 universal code，可以把 `n` bits 映射为：

```text
n + log₂ n + O(log log n) bits
```

并且编码和解码算法满足：

```text
space = O(log n) bits
time = O(1) per operation / per word
```

这个结果的重要性在于，它同时满足三点：

```text
接近 Elias code 的最优 overhead；
支持在线处理；
只需要低空间。
```

与 Elias codes 相比，本文方法不需要一开始知道 `n`，因此适合在线场景。与朴素 block-marker 方法相比，本文方法不需要每个 block 额外付出一个 bit，因此避免了线性冗余。

从技术路线看，Theorem 2 是本文第二个核心结果。Day 3 主要理解它的问题动机；完整技术实现会在 Section 5 中通过 information carriers 和 slowly growing blocks 展开。

---

## 与 Day 4 SOLE Encoding 的关系

Section 1.2 之后，论文在 Section 2 先给出一个简化但非常重要的构造：SOLE prefix-free encoding。

SOLE 的输入模型是：

```text
输入按 b-bit blocks 到达；
B = 2^b；
普通 block 来自 [B]；
加入 eof 后，字母表变为 [B] ∪ {eof}，大小为 B + 1。
```

SOLE 要解决的问题正是 Day 3 得到的核心换基问题：

```text
把 [B + 1] 字母表上的流，重新编码成 [B] 字母表上的 blocks。
```

它并不是最终的 Theorem 2，但它展示了本文最关键的局部换基思想：不要为每个 EOF 可能性单独浪费一个完整 bit，而是把多个符号组合起来，通过局部可逆的算术变换，在相邻 blocks 之间重新分配信息量。

因此，Day 3 的内容主要回答“为什么需要 online prefix-free encoding”和“为什么这可以看作 base conversion 问题”；Day 4 的 SOLE encoding 则开始回答“具体如何做这种局部 base conversion”。

---

## 密码学中的 Prefix-Free Encoding 动机

论文特别强调，prefix-free encoding 不只是通信和存储中的自终止编码问题，也和密码学中很多迭代式构造的安全性有关。

许多密码学构造会先设计一个固定输入长度的压缩函数：

```text
f : {0,1}^s × {0,1}^b → {0,1}^s
```

其中：

```text
b = block length
s = buffer length / state length
```

然后把任意长消息：

```text
M = m₁ m₂ ... mₙ
```

拆成 `b`-bit blocks，并通过 cascade mode 逐步处理：

```text
x₀ = IV
xᵢ = f(xᵢ₋₁, mᵢ)
output y = xₙ
```

这里 `IV` 可以是固定常数，也可以是密钥，具体取决于构造类型。

这种构造的问题在于：如果一个消息是另一个消息的前缀，那么短消息的输出可能正好成为长消息继续计算的中间状态。例如：

```text
M₂ = M₁ M'
```

则 cascade 计算具有如下关系：

```text
Cascade(M₂, f, IV)
= Cascade(M', f, Cascade(M₁, f, IV))
```

这意味着，如果攻击者知道短消息 `M₁` 的输出，就可能继续处理后缀 `M'`，得到长消息 `M₂` 的相关输出。这类攻击通常称为 extension attack。

Prefix-free encoding 的作用就是消除这种前缀关系。若所有合法消息编码构成 prefix-free code，则不存在一个合法编码是另一个合法编码前缀的情况。这样，不同消息在 cascade 计算树中不会形成祖先和后代关系，从而避免利用短消息输出扩展成长消息输出。

---

## CBC-MAC 中的 Extension Attack

CBC-MAC 是从 block cipher 构造 message authentication code 的经典方法。设 block cipher 为：

```text
g_k(m)
```

其中 `k` 是密钥，`m` 是一个 `b`-bit block。CBC-MAC 可以看成 cascade mode 的一个例子，其压缩函数为：

```text
f_k(x, m) = g_k(x ⊕ m)
```

对消息：

```text
M = m₁ m₂ ... mₙ
```

CBC-MAC 的计算过程可以写成：

```text
CBC-MAC(k, M)
= g_k(mₙ ⊕ g_k(mₙ₋₁ ⊕ ... g_k(m₁)))
```

论文指出，CBC-MAC 在消息编码 prefix-free 时是安全的；如果没有 prefix-free 约束，则容易受到攻击。

一个典型攻击是：

```text
y = CBC-MAC(k, m₁) = g_k(m₁)
```

攻击者可以构造第二个 block：

```text
m₂ = m₁ ⊕ y
```

于是：

```text
CBC-MAC(k, m₁m₂)
= g_k(m₂ ⊕ y)
= g_k((m₁ ⊕ y) ⊕ y)
= g_k(m₁)
= y
```

这说明攻击者在知道单块消息 `m₁` 的 MAC 后，可以伪造两块消息 `m₁m₂` 的 MAC。问题的根源是不同长度消息之间没有通过 prefix-free 编码隔离开。

常见工程修复方法是 encrypted CBC-MAC，即再使用一个新密钥 `k'` 对最终 CBC-MAC 结果加密。但这会增加密钥数量。论文的观点是：如果有高效 online prefix-free encoding，那么可以更自然地从编码层面避免 extension attack。

---

## Cascade PRF 中的 Extension Attack

Cascade PRF 用于把固定输入长度的 pseudorandom function 扩展到可变长度消息。设：

```text
g_k : {0,1}^b → {0,1}^s
```

是一个 keyed PRF。定义压缩函数：

```text
f(x, m) = g_x(m)
```

也就是说，当前 state `x` 被当作下一次调用 PRF 的 key。初始状态为真实密钥：

```text
x₀ = k
```

于是：

```text
F_k(M) = Cascade(M, f, k)
```

如果消息编码是 prefix-free 的，则任意两个不同消息不会形成前缀关系，它们在某个 block 处会分叉，后续 state 会变得随机且不相关。

但如果没有 prefix-free 约束，攻击非常直接。若：

```text
M₂ = M₁ M'
```

则：

```text
F_k(M₂)
= F_{F_k(M₁)}(M')
```

也就是说，知道 `M₁` 的 PRF 输出后，可以把它当成新的 key，继续计算 `M₂` 的 PRF 值。这破坏了 PRF 在可变长度输入上的伪随机性。

因此，cascade PRF 的安全性本质上依赖消息编码的 prefix-free 性质。

---

## Merkle-Damgård Hash Functions 中的 Extension Attack

Merkle-Damgård 是构造 cryptographic hash functions 的经典方式。它同样采用 cascade mode：给定压缩函数：

```text
f : {0,1}^{s+b} → {0,1}^s
```

和固定初始值 `IV`，对消息 blocks 逐个更新状态：

```text
H(M) = Cascade(M, f, IV)
```

这种结构的问题是 length extension attack。若攻击者知道：

```text
y = H(M)
```

即使不知道 `M` 的具体内容，也可以把 `y` 当成新的初始状态，继续处理任意后缀 `M'`：

```text
y' = Cascade(M', f, y)
```

这个结果对应某种扩展消息的 hash 值。对于理想随机预言机来说，知道 `H(M)` 不应该让攻击者计算出 `H(M, M')` 这样的相关值；但 plain Merkle-Damgård 的 cascade 结构会暴露这种可扩展性。

论文指出，最干净的解决方式之一是先对消息做 prefix-free encoding，再应用 cascade hash。这样合法消息之间不会形成前缀关系，攻击者不能把一个合法消息的 hash 输出自然地当作另一个合法消息的中间状态继续扩展。

在本文之前，由于 online PFE 被认为可能太浪费，许多 hash 和 MAC 构造采用了不同的 ad hoc workaround。本文的高效 online PFE 说明，可以用更统一的方式处理这类问题。

---

## Domain Extension of MACs

除了 CBC-MAC 和 cascade PRF，论文还讨论了 MAC 的 domain extension。设有一个固定输入长度的 MAC：

```text
f_k : {0,1}^{b+s} → {0,1}^s
```

可以定义：

```text
Cascade-MAC(k, M) = Cascade(M, f_k, 0^s)
```

这里同一个 key `k` 会在整个 cascade 过程中重复使用。论文指出，Maurer 和 Sjödin 已经观察到：如果 `M` 以 prefix-free 形式编码，则 Cascade-MAC 是安全的。

如果不使用 online PFE，则需要其他修复方法。例如额外引入一个 key `k'`，在最后再调用一次 MAC，或者使用其他更特殊的构造。论文认为，这些方法虽然可行，但通常不如“先做 prefix-free encoding，再做 cascade”直观。

因此，高效 online PFE 的意义在于：它降低了使用 prefix-free encoding 的代价，使 PFE-then-Cascade 重新成为一种简单、统一、有理论解释的 domain extension 方法。

---

## Multi-Property Preservation

论文最后还提到 multi-property preserving transform。许多密码学场景希望一个 transform 能同时保留多个安全性质，例如 collision resistance、pseudorandomness、unpredictability 等。

如果先对消息做 prefix-free encoding，再应用 cascade construction，那么很多前缀扩展类攻击会被统一排除。因此：

```text
PFE-then-Cascade
```

可以被看作一种简单的 multi-property preserving transform。

在本文之前，online PFE 被认为可能太浪费，所以相关工作需要设计更复杂的 MPPT。本文给出了高效 PFE 后，这种更直接的方案变得更有吸引力。

---

## 关键符号整理

```text
n
```

输入 bit stream 的总长度，在线场景中事先未知。

```text
b
```

block size。实际系统中可能取较大值，例如 128 bits。

```text
B = 2^b
```

一个普通 `b`-bit block 的字母表大小。

```text
[B]
```

表示集合 `{0, 1, ..., B - 1}`。

```text
eof
```

end-of-file symbol，用于标记输入结束。

```text
B + 1
```

加入 EOF 后的字母表大小。

```text
PFE
```

prefix-free encoding。

```text
Cascade(M, f, IV)
```

对消息 `M` 使用压缩函数 `f` 和初始向量 `IV` 做迭代处理。

```text
extension attack
```

利用一个消息是另一个消息前缀的结构，从短消息输出继续计算长消息相关输出的攻击。

---

## 关键公式整理

### Elias Code 长度

Elias delta-code 可以达到：

```text
n + O(log n)
```

Elias omega-code 可以达到：

```text
n + log₂ n + log₂ log₂ n + ... + O(log* n)
```

### 朴素 Block-Marker 方法冗余

每个 `b`-bit block 多加 1 bit 标记，总冗余为：

```text
Θ(n / b)
```

若 `b` 固定，则这是关于 `n` 的线性冗余。

### EOF 字母表扩大

普通 block 字母表：

```text
B = 2^b
```

加入 EOF 后：

```text
B + 1 = 2^b + 1
```

单个符号的信息量增加为：

```text
log₂(B + 1) - log₂ B
= log₂(1 + 1/B)
```

当 `B` 很大时，这远小于 1 bit。

### 理想 Base Conversion 下的 Overhead

若有 `n` 个 `b`-bit blocks，加入 EOF 后有 `n + 1` 个来自 `2^b + 1` 字母表的符号。理想编码长度为：

```text
(n + 1) log₂(2^b + 1)
```

原始输入长度为：

```text
n · b
```

overhead 为：

```text
(n + 1) log₂(2^b + 1) - n · b
```

在 `n ≤ 2^b` 的简化假设下，可控制为：

```text
b + O(1)
```

### Theorem 2 编码长度

本文最终给出的 online universal code 长度为：

```text
n + log₂ n + O(log log n)
```

空间和时间为：

```text
space = O(log n) bits
time = O(1) per operation
```
