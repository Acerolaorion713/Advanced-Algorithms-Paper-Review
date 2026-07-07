# Advanced Algorithms Paper Review

本项目用于完成论文 *Changing Base without Losing Space* 的阅读、理解与 review 写作。

## 论文信息

- Title: *Changing Base without Losing Space*
- Authors: Yevgeniy Dodis, Mihai Pătraşcu, Mikkel Thorup
- Date: April 11, 2010

## 项目目标

本项目的目标不是简单概述论文，而是完成一篇较完整、可核验的论文 review。最终 review 需要解释论文解决的问题、问题的重要性、相关研究背景、核心技术直觉、技术实现方式，以及正确性和优越性证明。

当前选择的论文有两个核心结果：

- 使用 `⌈n log₂ Σ⌉` bits 表示长度为 `n`、字母表大小为 `Σ` 的向量，并支持任意元素 `O(1)` 时间读写。
- 对在线到达且长度未知的 bit stream 构造长度为 `n + log₂ n + O(log log n)` 的 prefix-free encoding，并且编码、解码只需要 `O(log n)` bits 空间，按 word 处理为常数时间。

## 当前项目结构

```text
Advanced-Algorithms-Paper-Review/
├── README.md
├── paper/
│   └── Changing_base_without_losing_space.pdf
├── notes/
│   ├── day01_paper_overview.md
│   └── day02_succinct_data_structures.md
├── ai_logs/
│   ├── day01_ai_log.md
│   └── day02_ai_log.md
└── review/
    ├── main.tex
    └── sections/
        ├── 01_introduction.tex
        └── 02_problem_and_motivation.tex
```

## 文件夹说明

### notes

`notes/` 用于记录每天阅读论文的过程性理解。它不是最终提交的正式 review，但需要为正式 review 提供素材。每篇 notes 应该写清楚阅读范围、内容总结、论文具体解析、个人理解、可写入 review 的段落草稿，以及后续待验证问题。

### ai_logs

`ai_logs/` 用于记录 AI 辅助使用过程。它不是学习笔记，而是 AI 使用审计记录。每篇 ai log 应该记录当天为什么使用 AI、提出了什么 prompt、AI 回答了什么、哪些内容被采纳、哪些内容未采纳，以及人工核验情况。

### review

`review/` 用于保存正式 LaTeX review。`main.tex` 是主入口，`sections/` 中保存各章节内容。当前已经完成 introduction 和 problem/motivation 的初稿。

## 当前进度

### Day 1

完成论文 Abstract 和 Introduction 的总览阅读。

已完成文件：

- `notes/day01_paper_overview.md`
- `ai_logs/day01_ai_log.md`
- `review/sections/01_introduction.tex`

主要内容：

- 论文两个核心问题与两个主要定理。
- 向量表示问题的基本矛盾。
- 在线 prefix-free encoding 的问题背景。
- arithmetic coding 与本文局部换基目标的区别。
- 后续技术路线：SOLE encoding、information carriers、vector representation、online prefix-free code。

### Day 2

完成 Section 1.1 和 Section 1.3 的阅读，重点整理 succinct data structures 背景和向量表示问题动机。

已完成文件：

- `notes/day02_succinct_data_structures.md`
- `ai_logs/day02_ai_log.md`
- `review/sections/02_problem_and_motivation.tex`

主要内容：

- 向量表示的信息论最优空间 `⌈n log₂ Σ⌉`。
- 整体 base conversion 为什么空间最优但局部性差。
- 逐元素定长编码为什么局部访问好但产生线性冗余。
- succinct data structures 中 redundancy、local access 和 Word RAM 模型的意义。
- 旧结果与 Theorem 1 的差别。
- arithmetic coding 为什么不能满足 worst-case locality。

## 编译方式

进入 `review/` 目录后，可以使用 LaTeX 编译主文件：

```bash
pdflatex main.tex
```

如果后续加入 BibTeX 文献库，则需要按完整流程编译：

```bash
pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```

## 后续计划

接下来按天继续推进：

- Day 3：阅读 Section 1.2 和 Appendix A，整理 online prefix-free encoding 与密码学动机。
- Day 4：阅读 Section 2，整理 SOLE encoding。
- Day 5：阅读 Section 3，整理 information carrier lemma。
- Day 6：阅读 Section 4，整理 vector representation。
- Day 7：阅读 Section 5，整理 online prefix-free code。
- Day 8：整理 related work。
- Day 9：整理证明、复杂度和优越性核验。
- Day 10：整合详细版 review。
- Day 11：润色、压缩并生成精炼版 review。
