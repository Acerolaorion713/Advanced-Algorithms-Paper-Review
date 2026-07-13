# Day 8 Notes: Related Work

## 阅读与检索范围

本次阅读范围包括论文 *Changing Base without Losing Space* 的 Introduction、Section 1.1--1.3、References 与 Appendix A，并回顾 Day 1--Day 7 已完成的技术章节。外部检索以论文正式出版信息、作者主页、DBLP、arXiv 原始页面及公开论文全文为主。

今日目标是建立可供正式 review 使用的 related-work 比较框架，而不是简单复制原论文的参考文献列表。本日对应的正式文件是：

```text
review/sections/07_related_work.tex
```

本章与前七日的关系是：Day 1--Day 7 已经回答“论文做了什么、如何做到”，Day 8 进一步回答“这些结果建立在什么背景上、与相邻结果有何边界、后续研究应如何谨慎定位”。

------

## 内容总结

本论文横跨两条研究主线：succinct representation with local access，以及 online prefix-free coding。二者由同一个 local base-conversion / information-carrier primitive 连接，但比较对象不同。向量结果应与 redundancy、rank/select 和 cell-probe lower bounds 比较；在线结果应与 Elias universal codes、arithmetic coding 以及 cryptographic domain extension 比较。

论文的核心突破不是“压缩得比所有已有方法都好”，而是在均匀有限 alphabet、Word RAM 和特定局部操作下消除整数取整损失。它对长度为 $n$ 的向量达到精确的：

$$
\left\lceil n\log_2\Sigma\right\rceil
$$

bits，并支持常数时间读写；同时，它对未知最终长度的在线 bit stream 达到：

$$
n+\log_2n+O(\log\log n)
$$

bits、$O(\log n)$ 工作空间和每个 word/block operation 常数时间。

related work 的组织原则是按研究问题分类，分别说明 foundational background、direct predecessor、lower-bound context、comparison baseline、cryptographic motivation 与 subsequent related development，避免把“引用论文”自动写成“继承论文技术”。

------

## Related Work 的组织原则

1. 按问题而不是按年份组织：succinctness、locality、universal coding 和 domain extension 各有不同评价维度。
2. 区分模型：Word RAM、cell-probe、bit-probe、sequential streaming 和 random-access compression 的结论不可直接互换。
3. 区分关系强度：背景相近不等于直接延伸；引用原论文也不自动说明使用其 information-carrier construction。
4. 区分上界与下界的适用对象：rank/select lower bound 不会因为向量 coordinate access 存在 exact-space upper bound 而被推翻。
5. 对 post-2010 工作只保留能够从原论文全文和正式元数据同时核验的结果。

------

## Succinct Data Structures 与 Redundancy

长度为 $n$、alphabet size 为 $Sigma$ 的向量共有 $Sigma^n$ 种取值，因此信息论下界为：

$$
\left\lceil\log_2\Sigma^n\right\rceil
=
\left\lceil n\log_2\Sigma\right\rceil.
$$

若逐元素使用固定宽度：

$$
n\left\lceil\log_2\Sigma\right\rceil,
$$

则浪费：

$$
n\left(\left\lceil\log_2\Sigma\right\rceil-\log_2\Sigma\right)
$$

bits。当 $Sigma$ 不是 2 的幂时，该冗余通常为线性。若将整个向量看成一个 $Sigma$ 进制大整数，空间达到最优，但一次 coordinate read/write 会变成全局换基。

Jacobson 的工作奠定了 succinct data structures 的目标：接近 information-theoretic optimum，同时仍支持结构查询。Golynski 等人的 succinct indices 与 Pătraşcu 的 `Succincter` 进一步降低了 redundancy 并研究 time-space tradeoff。原论文在普通 finite-alphabet vector 上达到的是精确整数界，而不是一般形式的“所有 succinct structures 都能零冗余”。

------

## Rank/Select、Succinct Indexes 与 Lower Bounds

rank/select 是许多 succinct trees、strings 和 indexes 的导航基础。Gal--Miltersen、Golynski、Pătraşcu--Viola 与 Viola 分别在 cell-probe 或 bit-probe 等模型中给出 redundancy lower bounds。

这些 lower bounds 具有明确的 problem definition、query set、word/probe model 和 encoding assumptions。原论文的 vector problem 只要求：

```text
read A[i]
write A[i]
```

它不要求：

```text
rank(a, i)
select(a, j)
set membership
predecessor
```

因此论文不是推翻 rank/select lower bounds，而是展示 abstract vector problem 可以不用 rank/select machinery。该结果支持“可能存在另一条技术路线”这一判断，但不能自动推广到 dictionary 或 richer succinct queries。

------

## Arithmetic Coding 与 Locality

Arithmetic coding 针对独立分布 $D$ 可以实现接近：

$$
nH(D)
$$

的压缩，并适用于 nonuniform distributions。Witten--Neal--Cleary 与 Moffat--Neal--Witten 的 bounded-precision streaming 实现使用 renormalization 保持有限精度。

困难出现在 interval 跨越 $1/2$ 时。算法可能暂时不知道下一输出 bit，只能累计 outstanding bits；后续 interval 落到一侧后，再 burst 输出一串 bits。这意味着一个位置的编码可能依赖远处字符，最坏情况下不支持论文所需的局部随机访问、局部修改和稳定 per-block output。

| Dimension | Arithmetic coding | This paper |
|---|---|---|
| Distribution | General, including biased sources | Uniform finite alphabet |
| Compression objective | Near $nH(D)$ | Exact/near-exact base conversion |
| Streaming | Standard bounded-precision streaming | Online block processing |
| Worst-case locality | Standard form does not provide it | Explicit local decode/update guarantees |
| Random access/update | Potentially nonlocal | Constant number of regions for vector operations |

正确结论不是 arithmetic coding “更差”，而是目标不同：它覆盖更广的分布模型，而原论文用 uniform-alphabet 限制换取 worst-case locality。

------

## Universal Codes 与 Online Prefix-Free Encoding

Elias delta、omega 等 universal codes 是经典的 offline self-delimiting 方法。已知长度时，可以把长度的自定界表示写在 payload 前。Elias omega 的迭代对数表达式比简单的：

$$
n+\log_2n+O(\log\log n)
$$

更精细。

在线未知长度时，encoder 在输出开头时尚不知道 $n$。固定 block EOF flag 虽可在线处理，却产生 linear redundancy。Maurer--Sjödin 在 AIL-MAC / FIL-MAC 背景中讨论了在线 PFE 的代价，并提出小空间可能需要线性冗余的 conjecture。原论文给出上述 $n+\log_2n+O(\log\log n)$ 结果，从而反驳该 conjecture。

| Method | Online | Length known in advance | Working memory | Redundancy above $n$ |
|---|---|---|---|---|
| Elias-type length header | No | Yes | Small | Iterated-log length description |
| Fixed-block EOF flag | Yes | No | One block | Linear in block count |
| Maurer--Sjödin context | Online target | No | Small target | Linear cost conjectured necessary |
| This paper | Yes | No | $O(\log n)$ bits | $\log_2n+O(\log\log n)$ |

------

## Cryptographic Domain Extension

相关密码学构造可统一写为：

```text
variable-length message
        ↓ split into blocks
iterative cascade state
        ↓
ancestor/descendant relation for prefix messages
        ↓
extension attack
        ↓
prefix-free encoding or construction-specific remedy
```

CBC-MAC、cascade PRF、GGM、Merkle--Damgård、NMAC/HMAC 与 multi-property-preserving transforms 不是本论文发明的。它们说明了同一个设计问题：若短消息编码是长消息编码的 prefix，短消息处理后的内部状态可成为继续处理 suffix 的起点。

Prefix-free encoding 排除这种祖先/后代关系，是一个通用、易解释的 remedy。其他方案可能使用额外 key、outer application、finalization、strengthening 或专门 transform。本论文的贡献是降低 online PFE 的代价，使 `PFE-then-cascade` 更有吸引力；Appendix A 是应用动机，不是主 base-conversion theorem，也不能替代对具体 primitive 与 security model 的分析。

------

## 论文发表后的相关研究

### Huacheng Yu, 2019：Optimal Succinct Rank Data Structure via Approximate Nonnegative Tensor Decomposition

- Venue：STOC 2019。
- 问题：succinct rank 的 redundancy/query-time tradeoff。
- 主要结果：在广泛参数范围内匹配 Pătraşcu--Viola cell-probe lower bound，并使用 approximate nonnegative tensor decomposition 构造上界。
- 与本论文关系：回应 2010 论文讨论的 rank barrier，但研究问题与技术路线不同。
- 关系分类：subsequent related development，非已证实的 direct extension。
- 是否进入正式 review：是，用于说明 rank/select 主线继续发展。

### Huacheng Yu, 2020/2022：Nearly Optimal Static Las Vegas Succinct Dictionary

- Venue：STOC 2020；SIAM Journal on Computing 2022 完整版。
- 问题：接近信息论最优空间的 static dictionary 与 constant expected query time。
- 主要结果：给出接近最优的 randomized succinct dictionary；建立 fractional-length strings 的 concatenation、fusion 等工具。
- 与本论文关系：fractional-length strings 与 information carriers 都体现“先累积 fractional information，再统一 rounding”的思想。
- 重要边界：该论文将工具明确追溯到 Pătraşcu 2008 的 spillover representation；不能仅凭相似性声称它直接继承 Dodis--Pătraşcu--Thorup 2010 构造。
- 关系分类：substantively related technique family。
- 是否进入正式 review：是，但使用保守表述。

### 未进入正式 review 的候选

- 一般 locally decodable source coding：分布、错误模型和 query guarantees 与本文不同，未核验到同模型直接延伸。
- 一般 dynamic succinct sequences：多数依赖 rank/select/aB-tree，不能仅凭 succinctness 归为本文后续。
- 仅在参考文献中引用 2010 论文的 graph/string data structures：citation 不足以证明技术继承。
- 2023 `Dynamic "Succincter"`：可核验其使用 spillover representation，但其明确 lineage 是 `Succincter`；为控制正式章节篇幅，保留在检索记录而未加入 bibliography。

------

## 文献核验表

| BibTeX key | Year | Topic | Main result / role | Relation to this paper | Source checked | Used in review |
|---|---:|---|---|---|---|---|
| `dodis2010changing` | 2010 | Local base conversion | Two main theorems | primary paper | paper PDF, DBLP, ACM metadata | yes |
| `jacobson1989space` | 1989 | Succinct structures | Foundational succinct representation | foundational background | paper references, DBLP | yes |
| `golynski2007size` | 2007 | Succinct indices | Reduced redundancy | direct predecessor context | paper PDF, DBLP | yes |
| `patrascu2008succincter` | 2008 | Succinct recursion | Time--redundancy framework | direct predecessor | paper PDF, DBLP | yes |
| `gal2003cell` | 2003 | Lower bounds | Cell-probe context | lower-bound context | paper PDF | yes |
| `golynski2009cell` | 2009 | Lower bounds | Succinct cell-probe bounds | lower-bound context | paper PDF, DBLP | yes |
| `patrascu2010cell` | 2010 | Rank/partial sums | Rank-related tradeoff lower bound | comparison boundary | paper PDF, DBLP | yes |
| `viola2009bit` | 2009 | Bit-probe | Vector/succinct bit-probe bound | model comparison | paper PDF, DBLP | yes |
| `witten1987arithmetic` | 1987 | Arithmetic coding | Bounded-precision coding basis | comparison baseline | paper PDF references | yes |
| `moffat1998arithmetic` | 1998 | Arithmetic coding | Implementation analysis | comparison baseline | paper PDF references | yes |
| `elias1975universal` | 1975 | Universal coding | Self-delimiting integer codes | offline baseline | publication metadata | yes |
| `maurer2005single` | 2005 | MAC/PFE | Online PFE conjecture context | conjecture refuted | paper PDF, DBLP metadata | yes |
| `bellare1994security` | 1994 | CBC-MAC | Prefix-free security context | cryptographic motivation | paper PDF references | yes |
| `bellare1996cascade` | 1996 | Cascade PRF | Variable-domain PRF construction | cryptographic motivation | paper PDF references | yes |
| `coron2005merkle` | 2005 | Hash extension | Domain extension transforms | cryptographic motivation | paper PDF references | yes |
| `yu2019rank` | 2019 | Succinct rank | Later rank tradeoff improvement | related, not direct | paper PDF, arXiv, DBLP | yes |
| `yu2022dictionary` | 2022 | Succinct dictionary | Fractional-length strings and dictionary | related technique family | journal paper, arXiv, DBLP | yes |



