# Day 9 Notes: Correctness, Complexity, and Proof Checklist

## 1. 阅读与核验范围

本次核验覆盖论文 *Changing Base without Losing Space* 的 Section 2--5：Section 2（PDF 第 5--9 页）给出 SOLE local transform，Section 3（PDF 第 9--11 页）抽象 information-carrier lemma，Section 4（PDF 第 11--13 页）证明 optimal-space vector representation，Section 5（PDF 第 13 页）证明 online prefix-free encoding。辅助回顾的正式章节为：

```text
review/sections/03_sole_encoding.tex
review/sections/04_information_carriers.tex
review/sections/05_vector_representation.tex
review/sections/06_online_prefix_free_encoding.tex
```

今日目标不是引入新算法，而是检查每个有限集合映射的定义域、值域、逆变换、局部恢复义务、取整误差、终止时序和 Word RAM 假设，并把这些义务组织成一条可追踪的证明链。对应正式文件是：

```text
review/sections/08_correctness_and_analysis.tex
```

------

## 2. 今日总体结论

论文的四个技术层次共享同一个证明模式：先把一个非二进制有限集合中的状态与另一个局部值组成乘积空间，再用商余数分解把其中一部分提交到整数个 bits，并把未提交部分作为 spill 传给一个受控的相邻位置。SOLE 是最具体的实例：Pass 1 制造交替的 short/long ranges，Pass 2 通过错位配对让两个 range 的乘积落入 $B^2$。Information carrier 将该技巧抽象为 $(x,y)\mapsto(m,s)$，其中旧 spill $y$ 必须只由 $m$ 恢复；这不是普通 injectivity 的附带结果，而是阻止访问沿 spill 链递归的关键。

两个主定理分别用不同拓扑组织该局部原语。向量表示把 carriers 放进隐式二叉树，同层同类节点共享参数；查询只访问节点和父节点，更新只重算节点和父节点，因此树高虽为 $O(\log n)$，操作时间仍为 $O(1)$。在线 PFE 则把 carriers 串成流，以 slowly growing blocks 使每步 $O(2^{-k_i})$ 的损失按尺度形成调和和，并用 3-block lag 把 EOF 放到 $N-2$，保证 decoder 在请求不存在的输出前已经看到终止。最容易误解的三点是：把“常数局部性”误写成“只依赖一个 block”；把 $n\log_2\Sigma+o(1)$ 直接取整成精确界；以及认为“添加 EOF”本身就自动证明 prefix-free。Day 9 的作用是把这些分散论证变成一条带边界条件与模型假设的完整 proof audit。

------

## 3. 证明依赖关系

| 证明义务 | 直接依赖 | 核验结论 |
|---|---|---|
| SOLE Pass 1 合法性 | $(B+1)^2\leq(B-4i)(B+4i+4)$ | 由 $B\geq2n^2$ 与 $2i+1\leq n$ 推出 |
| SOLE Pass 1 可逆性 | 两次 Euclidean division | ((a,q)) 唯一恢复 (z,x,y) |
| SOLE Pass 2 可逆性 | $(B+4i)(B-4i)<B^2$, $i\geq1$ | mixed-radix integer 唯一写成两个 base-$B$ digits |
| SOLE locality | Property 3 | 正反向各访问 4 个相邻 blocks，而非 1 个 |
| carrier injectivity | $m=yC+c$, $s=\lfloor x/C\rfloor$ | $m$ 恢复 $y,c$，再由 $sC+c$ 恢复 $x$ |
| old spill local recovery | $y=\lfloor m/C\rfloor$ | 不读取新 spill $s$ |
| carrier redundancy | $R_1=O(1/C)$, $R_2=O(C/X)$ | $C=\Theta(\sqrt X)$ 后为 $O(1/\sqrt X)$ |
| vector total redundancy | $X=\Theta(n^2)$, $N=O(n/\log_\Sigma n)$ | $N\,O(1/n)=o(1)$ |
| vector (O(1)) query | parent memory + node memory | 不沿树路径递归 |
| vector (O(1)) update | outgoing spill depends on carrier, not old-spill value | 变化在 parent 被吸收，不到 grandparent |
| exact-space rounding | inverse-polynomial distance to nearest integer | 把 $O(n^{-c})$ 压到整数间隙以下 |
| online EOF correctness | EOF in input block (N-2) | 读 output block (N-1) 后即可发现 |
| online overhead | scale count $O(2^K/K)$ | $\sum_{K\leq\log n}O(1/K)=O(\log\log n)$ |
| streaming working memory | current block, spill, counters, lag/tail metadata | 常数个 $O(\log n)$-bit objects |

------

## 4. SOLE Encoding 证明核验

### 4.1 参数和定义域

统一使用：

$$
B=2^b,
\qquad
[B]=\{0,1,\ldots,B-1\}.
$$

普通输入 block 属于 ([B])。把 (mathsf{eof}) 标识为值 (B) 后，扩大的输入 alphabet 是 ([B+1])。Basic SOLE 假设数据 block 数 (n) 为奇数，且：

$$
b\geq2\log_2n+1,
\qquad
B\geq2n^2.
$$

### 4.2 Pass 1 的范围证明

对 zero-based pair index $i\geq0$，令：

$$
(x,y)\in[B+1]\times[B+1],
\qquad
z=y(B+1)+x.
$$

因此 $0\leq z<(B+1)^2$。按 $B-4i$ 做 Euclidean division：

$$
a=z\bmod(B-4i),
\qquad
q=\left\lfloor\frac{z}{B-4i}\right\rfloor.
$$

余数定义立即给出 $a\in[B-4i]$。商的范围依赖容量不等式，必须展开为：

$$
\begin{aligned}
(B-4i)(B+4i+4)
&=B^2+4B-16i^2-16i,\\
(B-4i)(B+4i+4)-(B+1)^2
&=2B-16i(i+1)-1.
\end{aligned}
$$

所以：

$$
(B+1)^2\leq(B-4i)(B+4i+4)
$$

等价于：

$$
B\geq8i(i+1)+\frac12
=2(2i+1)^2-\frac32.
$$

由 $2i+1\leq n$ 和 $B\geq2n^2$ 得：

$$
B\geq2n^2\geq2(2i+1)^2
>2(2i+1)^2-\frac32.
$$

故：

$$
z<(B+1)^2\leq(B-4i)(B+4i+4),
$$

从而 $q<B+4i+4$，即 $q\in[B+4i+4]$。同一推导也保证除数 $B-4i>0$。

### 4.3 Pass 1 逆变换

给定 ((a,q))，先恢复：

$$
z=q(B-4i)+a.
$$

再按 (B+1) 分解：

$$
x=z\bmod(B+1),
\qquad
y=\left\lfloor\frac{z}{B+1}\right\rfloor.
$$

两次唯一性都来自 Euclidean division theorem：给定正除数，商与满足标准范围的余数唯一。因此 Pass 1 不只是容量足够，而是具有显式逆映射。

### 4.4 Pass 2

Pass 1 后错位 regroup。对 $i\geq1$，pair 属于：

$$
[B+4i]\times[B-4i].
$$

其状态数为：

$$
(B+4i)(B-4i)=B^2-16i^2<B^2.
$$

把 pair 合成一个 $[B^2]$ 中的整数，再按 $B$ 做商余数分解，得到唯一的两个 $[B]$ digits。严格小于号需要 $i\geq1$；位置 1 的 $[B]$ singleton 不进入这一 regroup 操作。

### 4.5 终止处理

令奇数数据长度 (n=2r+1)。EOF 位于位置 (n+1=2r+2)，即最后一个 Pass 1 pair 的第二项。该 pair 输出的 long range 是 ([B+4r+4])。算法再附加合法值：

$$
0\in[B-4r-4],
$$

使最后 long/short pair 可被 Pass 2 编成两个普通 blocks。Decoder 一旦逆变换得到 (mathsf{eof}) 就停止，不把人工 0 当作消息。

Basic SOLE 只直接陈述奇数 (n)。对偶数 (n)，论文允许用额外 EOF block padding 规约到该情形，因此 basic variant 最坏可能浪费 3 blocks；Section 2.3 的 tighter variant 用提前 EOF 和两个 EOF symbols 同时处理 parity，并达到 (n+1) blocks。Day 9 不重写该改进算法。

Property 3 给出：input blocks $2i+1,2i+2$ 可由 output blocks $2i,\ldots,2i+3$ 解码。对最后 EOF pair，所需窗口仍完全落在已经输出的末尾内。因此 decoder 发现 EOF 不要求探测文件末尾之外的数据；这一步才完成 self-delimiting / prefix-free 的终止证明。

### 4.6 局部性

论文的准确 locality statement 是：

```text
Output blocks 2i and 2i+1 depend on input blocks 2i-1,...,2i+2.
Input blocks 2i+1 and 2i+2 are decoded from output blocks 2i,...,2i+3.
```

正向与反向均为 4-block window。结论是 (O(1)) locality，不是“一个 output 只依赖一个 input”。

------

## 5. Information-Carrier Lemma 证明核验

`review/sections/04_information_carriers.tex` 的构造与论文一致。统一符号如下：

$$
x\in[X],
\qquad
y\in[Y].
$$

选择 (M)，定义：

$$
C=\left\lfloor\frac{2^M}{Y}\right\rfloor,
\qquad
S=\left\lceil\frac{X}{C}\right\rceil,
$$

并要求 $C\geq1$。分解 carrier：

$$
c=x\bmod C,
\qquad
s=\left\lfloor\frac{x}{C}\right\rfloor,
$$

然后提交旧 spill：

$$
m=yC+c.
$$

逐项核验：

1. $0\leq c<C$，且 $0\leq y<Y$，故 $0\leq m<YC\leq2^M$。
2. 因此 $m\in[2^M]$，是合法的 $M$-bit memory content。
3. 因为 $m=yC+c$ 且 $c<C$，$y=\lfloor m/C\rfloor$。
4. 同理 $c=m\bmod C$。
5. 因为 (x=sC+c)，由 ((m,s)) 恢复 (x)。
6. 恢复 (y) 只读取 (m)，不读取 (s)。
7. 恢复 (x) 使用 (m) 给出的 (c) 和新 spill (s)。
8. 两个输入均被唯一恢复，因此 ((x,y)\mapsto(m,s)) injective。

其中第 6 项是局部访问的核心。如果 old spill 还依赖 (s)，查询可能需要继续追踪后继 spill，局部性即失效。

------

## 6. Lemma 4 的冗余推导

### 6.1 第一项 $R_1$

由 floor 定义：

$$
C\leq\frac{2^M}{Y}<C+1,
$$

所以：

$$
YC\leq2^M<Y(C+1).
$$

因此：

$$
\begin{aligned}
R_1
&=M-\log_2(YC)\\
&=\log_2\frac{2^M}{YC}\\
&<\log_2\left(1+\frac1C\right)\\
&\leq\frac{1}{C\ln2}
=O\left(\frac1C\right).
\end{aligned}
$$

`review/sections/04_information_carriers.tex` 中的 (2^M<Y(C+1)) 是严格成立的，包括 (2^M/Y) 恰为整数的情况。

### 6.2 第二项 $R_2$

由 ceiling 定义：

$$
S<\frac XC+1,
\qquad
CS<X+C.
$$

所以：

$$
\begin{aligned}
R_2
&=\log_2C+\log_2S-\log_2X\\
&=\log_2\frac{CS}{X}\\
&<\log_2\left(1+\frac CX\right)\\
&\leq\frac{C}{X\ln2}
=O\left(\frac CX\right).
\end{aligned}
$$

### 6.3 平衡与参数范围

总冗余：

$$
R_1+R_2
=O\left(\frac1C+\frac CX\right).
$$

令 $1/C\asymp C/X$，得到：

$$
C=\Theta(\sqrt X),
\qquad
R_1+R_2=O\left(\frac1{\sqrt X}\right).
$$

此外：

$$
S=\left\lceil\frac XC\right\rceil=O(\sqrt X),
$$

且由 $2^M<Y(C+1)$：

$$
2^M=O(Y\sqrt X).
$$

所有 floor、ceiling 与严格不等号均保留；这些界是 state-count ratio 的对数，而不是把 fractional bits 当作可单独寻址的物理 bits。

------

## 7. Theorem 1：Vector Representation

### 7.1 分块

本节 $\Sigma$ 表示 alphabet cardinality，而不是集合对象。对固定 $\Sigma\geq2$，选择：

$$
X=\Theta(n^2).
$$

每个 carrier 含 $\Theta(\log_\Sigma n)$ 个原始 coordinates，carrier 数为：

$$
N=O\left(\frac{n}{\log_\Sigma n}\right).
$$

$\Sigma=1$ 是零信息的平凡边界；$\Sigma$ 为 2 的幂时，普通 fixed-width packing 已无逐 coordinate 取整损失。

### 7.2 顺序方案

每步冗余：

$$
O(1/\sqrt X)=O(1/n).
$$

累计 carrier 冗余：

$$
N\cdot O(1/n)
=O\left(\frac1{\log_\Sigma n}\right)
=o(1).
$$

最终 spill 的独立向上取整最多再带来 1 bit。顺序方案的主要缺陷不是空间或局部访问，而是不同 $Y_i,M_i$ 和 offsets 可能需要 $O(N)$ 组预计算 constants。

### 7.3 树形方案

在 heap-ordered implicit tree 的同一层，按子树高度只有三类节点：

1. 一段高度为 (k) 的完整子树；
2. 至多一个不完整子树；
3. 剩余高度为 (k-1) 的完整子树。

同层同类节点的两棵 child subtrees 同构；由底向上的归纳，它们收到相同 spill universe sizes，故共享 $M,C,S$、memory-field size 和定位规则。每层常数类、树高 $O(\log n)$，所以只需 $O(\log n)$ 组预计算 word constants。

### 7.4 查询为何是 (O(1))

恢复非根节点 (v) 的 carrier 只需：

1. 读取 parent((v)) 的 memory bits，恢复 parent 的 combined old spill，并从中取出 (v) 的 outgoing spill；
2. 读取 $v$ 自己的 memory bits，与该 spill 一起恢复 $x_v$；
3. 从 $x_v$ 中提取目标 coordinate。

根节点使用单独保存的 root spill。该过程访问常数个局部区域，不沿 root-to-leaf 或 leaf-to-root path，因此不是 $O(\log n)$。

### 7.5 更新为何不传播

修改 $x_v$ 后重算 $v$，其 outgoing spill 可能改变，于是 parent 的 combined old spill 改变，必须重算 parent memory。关键是 carrier mapping 的新 spill 为：

$$
s=\left\lfloor\frac{x}{C}\right\rfloor,
$$

只依赖该节点 carrier (x) 与预计算参数 (C)，不依赖 old-spill 的具体值。Parent 自己的 carrier 未改变，所以 parent outgoing spill 不变；grandparent 无需更新。根节点只改 root memory/root spill，叶节点以固定 null child spills 开始，不完整节点使用其已知的 singleton missing-child range，均不改变常数局部性。

### 7.6 精确空间

必须区分：

$$
n\log_2\Sigma+o(1),
\qquad
\left\lceil n\log_2\Sigma+o(1)\right\rceil,
\qquad
\left\lceil n\log_2\Sigma\right\rceil+1.
$$

前两者不能自动推出精确 $\lceil n\log_2\Sigma\rceil$。论文把 carriers 增大，使累计误差成为任意固定 $c$ 下的 $O(n^{-c})$，同时只需常数个 $O(\log n)$-bit words。若 $\Sigma$ 不是 2 的幂，linear forms in logarithms 给出 $n\log_2\Sigma$ 到最近整数的 inverse-polynomial 非零下界；选足够大的 $c$，误差小于到下一个整数的间隙，最终 ceiling 不增加额外一位。若 $\Sigma$ 是 2 的幂，信息量本来就是整数且 fixed-width 表示直接达到精确界。这里不自行证明 Baker--Wüstholz 定理，只核验它在论文中的用途。

------

## 8. Theorem 2：Online Prefix-Free Encoding

### 8.1 变量命名

本 notes 使用：

- (t)：决定下一个 block 大小时已经读入的 bit 数；
- (n)：最终输入长度；
- (N)：最终 block 总数；
- $k_i$：第 $i$ 个完整 block 的 half-block 参数。

初始常数规模的 blocks 可单独处理，不影响渐近界。随后：

$$
2k_i=2\lceil\log_2t\rceil.
$$

### 8.2 Slowly growing blocks

每个完整 block 由两个 $k_i$-bit halves 组成，其 universe 为：

$$
X_i=2^{2k_i}=\Theta(t^2)
$$

（在同一尺度内取常数因子意义）。因此 carrier redundancy 为：

$$
O(1/\sqrt{X_i})=O(2^{-k_i}).
$$

### 8.3 EOF alphabet enlargement

第二 half alphabet 从 $[2^{k_i}]$ 扩大到 $[2^{k_i}]\cup\{\mathsf{eof}\}$。额外信息量是：

$$
\begin{aligned}
&\log_2\bigl(2^{k_i}(2^{k_i}+1)\bigr)-2k_i\\
&=\log_2(1+2^{-k_i})
=O(2^{-k_i}).
\end{aligned}
$$

### 8.4 Harmonic sum

固定 $k_i=K$ 时，当前规模约在 $2^{K-1}<t\leq2^K$，该尺度含 $O(2^K)$ bits，而每个完整 block 长 $\Theta(K)$，所以 block 数为：

$$
O(2^K/K).
$$

该尺度的 alphabet 或 carrier cost 均为：

$$
O(2^K/K)\cdot O(2^{-K})=O(1/K).
$$

最终 $t\leq n$，故最大尺度 $K\leq\lceil\log_2n\rceil$。于是：

$$
\sum_{K=1}^{\lceil\log_2n\rceil}O(1/K)
=O(\log\log n).
$$

### 8.5 提前 EOF 与 3-block lag

Encoder 缓存最后常数个 $O(\log n)$-bit blocks，形成 3-block lag；它不回写已输出数据。输入结束后，若完整 block $N-2$ 原为 $(a,b)$，则输出逻辑值 $(a,\mathsf{eof})$，并把 displaced half $b$ 原样放进特殊 tail。最后不完整 block $N$ 使用 Elias code 单独处理。Lag buffer、tail metadata 与最后 partial block 合计仍为 $O(\log n)$ bits。

### 8.6 Decoder termination

EOF 位于 input block $N-2$。Sequential carrier inverse 在读取 output block $N-1$ 后，已经得到解码 input block $N-2$ 所需的新 spill，因此可以恢复 $(a,\mathsf{eof})$。Decoder 立即停止普通 carrier parsing，转而解析 self-delimiting tail、恢复 $b$ 与 block $N$。所以它在请求任何不存在的 ordinary output block 之前发现 EOF；这才证明不同合法 codewords 不会形成无法判定的 prefix 关系。

### 8.7 总 overhead

| 来源 | 成本 | 说明 |
|---|---:|---|
| payload | (n) | 原始数据本身 |
| displaced half / early EOF | $\leq\log_2n$ | 同一机制，不重复计两个 $\log n$ 项 |
| alphabet enlargement | $O(\log\log n)$ | 分尺度调和和 |
| carrier rounding | $O(\log\log n)$ | 同一分尺度分析 |
| final incomplete block | $O(\log\log n)$ | 仅对 $O(\log n)$-bit tail 做 Elias coding |

因此：

$$
n+\log_2n+O(\log\log n).
$$

该结果是 near-optimal / within $O(\log\log n)$ of the Elias-scale bound，不是 Elias omega 精确 iterated-log 展开的等号。

### 8.8 时间和工作空间

工作空间包括 current block、current spill、block-size counters、3-block lag buffer 与 termination metadata，均为常数个 $O(\log n)$-bit 对象，总计 $O(\log n)$ bits。每个完整 block 使用常数次加、乘、除与 quotient--remainder 操作；正确表述是 Word RAM 上 $O(1)$ per word/block operation，而不是 $O(1)$ per individual bit。

------

## 9. 复杂度总表

| 构造 | Encoded size | Access / operation time | Working memory | Precomputed constants | Locality | Model |
|---|---|---|---|---|---|---|
| Basic SOLE | odd $n$: $n+2$ $b$-bit blocks | $O(1)$ per double-block; random read/write $O(1)$ | $O(1)$ blocks | arithmetic progression parameters | 4-block windows | word/block arithmetic, $B\geq2n^2$ |
| Information carrier | $M+\log_2S=\log_2X+\log_2Y+O(1/\sqrt X)$ | $O(1)$ encode/decode | constant words | $M,C,S$ for fixed $X,Y$ | old spill from $m$ alone | Word RAM division/multiplication |
| Vector representation | $\lceil n\log_2\Sigma\rceil$ bits | $O(1)$ query/update | representation uses random access; operation uses constant words | $O(\log n)$ word constants | node + parent | Word RAM, $w=\Omega(\log n)$ |
| Online PFE | $n+\log_2n+O(\log\log n)$ bits | $O(1)$ per word/block | $O(\log n)$ bits | current-scale constants/counters | sequential one-block carrier dependency plus constant lag | streaming Word RAM |

------

## 10. 模型假设

- Memory 是 random-access Word RAM，word size $w=\Omega(\log n)$，可容纳 index 与 pointer-sized quantities。
- 论文只需要标准 unit-cost addition、multiplication、division；modulo 由 quotient--remainder 同步得到。
- 每个局部 carrier 及其参数装入常数个 words；exact-space refinement 使用 $O(c\log n)$-bit integers，其中 $c$ 是固定常数，因此仍为常数 words。
- 固定 universe parameters 可预计算；vector theorem 明确计入 $O(\log n)$ 个 precomputed word constants。
- Vector 结论依赖 random access；online PFE 结论依赖 block-level sequential processing。
- Alphabet 是 uniform finite alphabet；论文未给出 biased source 的 locally decodable arithmetic code。

这些是理论模型下的 worst-case asymptotic guarantees，不等于论文已在所有 CPU、division latency、cache hierarchy 或 cryptographic implementation 上做过 benchmark。

