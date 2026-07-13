# Day 8 AI Log

## 今日任务背景

今天的工作集中在 *Changing Base without Losing Space* 的 related work、参考文献体系与正式 review 接入。阅读范围包括原论文 Introduction、References、Appendix A，以及可核验的外部出版记录和 post-2010 原始论文。

AI 主要用于生成检索候选、建立比较维度、检查模型边界和整理 BibTeX。AI 输出不被视为独立事实来源；所有写入正式章节的文献事实均需由原论文、正式出版记录或候选论文全文核验。

------

## AI 使用记录

## Prompt 1：Related Work 的研究主线

**我提出的问题：**

> 请把论文的 related work 按研究问题组织，而不是按年份罗列，并区分向量表示和在线 prefix-free encoding 的比较对象。

**使用 AI 的目的：**

建立 Day 8 notes 与正式章节的共同骨架。

**AI 回答摘要：**

AI 建议分为 succinct structures and redundancy、rank/select and lower bounds、arithmetic coding、universal codes、cryptographic domain extension、subsequent developments 与 overall positioning。

**采纳内容：**

采纳主题式结构，并让两个主要 theorem 分别进入正确的文献语境。

**未采纳或暂不使用内容：**

未采用按年份从 1989 排到 2023 的流水账写法。

**人工核验情况：**

已对照原论文 Section 1.1--1.3 与 Appendix A，确认这些主题均由原文明确支持。

------

## Prompt 2：Rank/Select Lower Bounds 的边界

**我提出的问题：**

> 原论文的 exact-space vector theorem 是否推翻 Pătraşcu--Viola 或其他 rank/select lower bounds？

**使用 AI 的目的：**

避免把不同 query problem 和模型混为一谈。

**AI 回答摘要：**

AI 指出 coordinate access 不等于 rank/select；lower bounds 针对特定问题和模型。论文展示的是 vector problem 可以避开 rank/select machinery。

**采纳内容：**

采纳“does not contradict; avoids the machinery”这一核心边界。

**未采纳或暂不使用内容：**

拒绝了“lower bounds are obsolete”与“all succinct data structures can use zero redundancy”等泛化。

**人工核验情况：**

已核对原论文 Introduction 第 1.1 节与 Pătraşcu--Viola 条目的问题定义。

------

## Prompt 3：Arithmetic Coding 比较

**我提出的问题：**

> 请比较 bounded-precision arithmetic coding 与论文 local base conversion，重点解释 outstanding bits、burst output 和 worst-case locality。

**使用 AI 的目的：**

形成不贬低 arithmetic coding、又能明确技术差异的比较。

**AI 回答摘要：**

AI 将 arithmetic coding 的优势定位为 nonuniform sources 与 near-entropy sequential compression；将限制定位为标准实现中的非局部依赖和 bursty output。

**采纳内容：**

采纳 distribution、objective、streaming、locality、random access/update 五维比较表。

**未采纳或暂不使用内容：**

未采用“arithmetic coding is inferior”或“arithmetic coding cannot stream”的错误表述。

**人工核验情况：**

已对照原论文 Section 1.3 及 Witten--Neal--Cleary、Moffat--Neal--Witten 的元数据。

------

## Prompt 4：Elias Codes 与 Maurer--Sjödin

**我提出的问题：**

> 请区分 Elias offline code、fixed-block EOF 与本文 online PFE，并准确描述 Maurer--Sjödin conjecture。

**使用 AI 的目的：**

防止把 $n+\log n+O(\log\log n)$ 写成 Elias omega 的精确最优展开。

**AI 回答摘要：**

AI 区分了已知长度 header、未知长度在线输出与 per-block integral flag。原论文结果与 Elias-scale optimum 相差 $O(\log\log n)$，同时提供 $O(\log n)$ 工作空间和每 word/block 常数时间。

**采纳内容：**

采纳上述区分与比较表。

**未采纳或暂不使用内容：**

未采用“本文等于 Elias omega optimum”或“Elias code 可直接在线输出未知长度 header”的说法。

**人工核验情况：**

已核对原论文 Section 1.2、Theorem 2 与 Maurer--Sjödin 参考条目。

------

## Prompt 5：Cryptographic Domain Extension

**我提出的问题：**

> 如何用一条统一主线概括 CBC-MAC、cascade PRF、Merkle--Damgård、NMAC/HMAC 与 domain-extension transforms，而不把章节写成密码学综述？

**使用 AI 的目的：**

把 Appendix A 压缩为与 online PFE 直接相关的动机。

**AI 回答摘要：**

AI 使用 variable-length input → iterative cascade → ancestor/descendant relation → extension attack → prefix-free remedy 的组织方式，并把 additional keys、outer application、finalization 等列为 alternative remedies。

**采纳内容：**

采纳该统一主线，并明确论文没有发明这些 primitive。

**未采纳或暂不使用内容：**

没有复写 Appendix A 的攻击公式，也没有声称 PFE 自动保证任何具体 construction 的安全性。

**人工核验情况：**

已逐段核对 Appendix A 的 CBC-MAC、cascade PRF、Merkle--Damgård 和 MAC domain extension。

------

## Prompt 6：Post-2010 候选筛选

**我提出的问题：**

> 检索 2010 年后与 spillover representation、fractional-length strings、succinct rank/dictionary 和 local decoding 相关的工作，并区分 direct extension 与 related development。

**使用 AI 的目的：**

发现候选并避免基于 citation count 推断技术关系。

**AI 回答摘要：**

AI 找到 Huacheng Yu 2019 的 succinct rank、2020/2022 的 succinct dictionary，以及 2023 `Dynamic "Succincter"`。原始论文显示后两者的 spillover lineage 明确追溯到 Pătraşcu 2008。

**采纳内容：**

正式 review 保留 Yu 2019 与 Yu 2022，分别标为 rank 主线的后续进展和 substantively related fractional-length technique。

**未采纳或暂不使用内容：**

未把任何候选写成对 Dodis--Pătraşcu--Thorup 2010 的直接延伸；`Dynamic "Succincter"` 只保留在 notes 的检索记录中。

**人工核验情况：**

已打开候选论文原文，检查 abstract、introduction、spillover/fractional-length 定义及 bibliography lineage，并用 DBLP 核验作者、年份和 venue。

------

## Prompt 7：BibTeX 与 Citation Audit

**我提出的问题：**

> 请按 surnameYearKeyword 规范整理正式章节实际使用的条目，并检查作者重音、标题大小写、venue、pages 与 citation keys。

**使用 AI 的目的：**

将占位 `references.bib` 转换为有限、可核验、实际引用的数据库。

**AI 回答摘要：**

AI 建议对 Pătraşcu、Sjödin、Damgård 使用 BibTeX 兼容重音命令，对 CBC-MAC、AIL-MAC、FIL-MAC、GGM 等专名加花括号保护。

**采纳内容：**

采纳统一 key 规范与特殊字符处理，并只加入正式 review 使用的条目。

**未采纳或暂不使用内容：**

未添加仅在搜索结果中出现但正文未使用的弱相关文献，也未填写未可靠核验的 DOI。

**人工核验情况：**

已将所有 `\cite{...}` key 与 `references.bib` 逐项对照；最终状态以 BibTeX 实际编译结果为准。

