# Advanced Algorithms Paper Review

本项目用于完成论文 *Changing Base without Losing Space* 的阅读、理解与 review 写作。

## 论文信息

- **Title:** *Changing Base without Losing Space*
- **Authors:** Yevgeniy Dodis, Mihai Pătraşcu, Mikkel Thorup

## 项目状态

**Status:** Final review completed; pending user approval and commit.

本项目历时 10 天（Day 1 -- Day 10），完成了论文的完整阅读、背景研究、技术分析和 review 撰写。

## 最终项目结构

```text
Advanced-Algorithms-Paper-Review/
├── README.md
├── paper/
│   └── Changing_base_without_losing_space.pdf
├── notes/
│   ├── day01_paper_overview.md
│   ├── day02_succinct_data_structures.md
│   ├── day03_prefix_free_encoding.md
│   ├── day04_sole_encoding.md
│   ├── day05_information_carriers.md
│   ├── day06_vector_representation.md
│   ├── day07_online_prefix_free_encoding.md
│   ├── day08_related_work.md
│   └── day09_proof_checklist.md
├── ai_logs/
│   ├── day01_ai_log.md
│   ├── day02_ai_log.md
│   ├── day03_ai_log.md
│   ├── day04_ai_log.md
│   ├── day05_ai_log.md
│   ├── day06_ai_log.md
│   ├── day07_ai_log.md
│   ├── day08_ai_log.md
│   └── day09_ai_log.md
└── review/
    ├── main.tex                      
    ├── review_condensed.tex          
    ├── references.bib
    ├── sections/
    │   ├── 01_introduction.tex
    │   ├── 02_problem_and_motivation.tex
    │   ├── 03_sole_encoding.tex
    │   ├── 04_information_carriers.tex
    │   ├── 05_vector_representation.tex
    │   ├── 06_online_prefix_free_encoding.tex
    │   ├── 07_related_work.tex
    │   ├── 08_correctness_and_analysis.tex
    │   ├── 09_strengths_limitations.tex
    │   └── 10_conclusion.tex
    └── build/
        └── main.pdf          
```

## Notes 说明

`notes/` 目录记录每一天的阅读与理解产出：

| 文件 | 内容 |
|------|------|
| day01_paper_overview.md | 论文总览、两个核心问题、主要结果 |
| day02_succinct_data_structures.md | Succinct data structures 背景整理 |
| day03_prefix_free_encoding.md | Prefix-free encoding 与密码学动机 |
| day04_sole_encoding.md | SOLE encoding 阅读与笔记 |
| day05_information_carriers.md | Information carrier lemma 阅读与笔记 |
| day06_vector_representation.md | Optimal-space vector representation 阅读与笔记 |
| day07_online_prefix_free_encoding.md | Online prefix-free encoding 阅读与笔记 |
| day08_related_work.md | Related work 整理与 bibliography 补充 |
| day09_proof_checklist.md | Correctness、complexity 和 proof checklist 核验 |

## AI Logs 说明

`ai_logs/` 目录记录每一天的 AI 辅助使用情况。每个 log 包含 AI 使用的目的、采纳情况和人工核验情况。详见各文件。

AI 只用于辅助阅读、解释、结构组织和 LaTeX 编译。所有关键定理、证明、复杂度分析和文献事实已经过人工核验。

## Review 说明

### 长版 Review

**文件:** `review/main.tex`（含 `review/sections/` 下的 10 个章节文件）
**PDF:** `review/review.pdf`

长版 review 的章节结构：

1. **Introduction** --- 论文概览、两个核心问题、主要结果、技术路线图
2. **Problem and Motivation** --- 问题形式化、全局 vs. 局部编码、Word RAM、算术编码局限、密码学动机
3. **SOLE Encoding** --- 第一遍 Short-Odd Long-Even、第二遍重组、termination、locality
4. **Information Carriers** --- Spill 问题、carrier 抽象、lemma 陈述与证明、冗余分析
5. **Optimal-Space Vector Representation** --- 符号分组、隐式二叉树、O(1) 读写、参数共享、exact-space rounding
6. **Online Prefix-Free Encoding** --- Slowly growing blocks、early EOF、carrier 链、总长度与资源分析
7. **Related Work** --- Succinct structures、rank/select lower bounds、arithmetic coding、universal codes、密码学、post-2010 work
8. **Correctness and Complexity Analysis** --- 证明架构、SOLE/lemma/vector/online PFE 证明义务、Word RAM 模型假设
9. **Strengths, Limitations, and Open Questions** --- 8 项 strengths、10 项 limitations、7 个 open questions
10. **Conclusion** --- 问题回顾、information carrier 作为统一核心、理论价值、适用范围、最终评价

## 编译方式

环境要求：LaTeX（pdflatex + bibtex）。

```bash
cd review

pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```

## Day 1--Day 10 工作流

| Day | 任务 | 产出 |
|-----|------|------|
| 1 | 论文全文阅读 | notes/day01, ai_logs/day01 |
| 2 | Succinct data structures 背景 | notes/day02, ai_logs/day02 |
| 3 | Prefix-free encoding 与密码学动机 | notes/day03, ai_logs/day03 |
| 4 | SOLE encoding | notes/day04, ai_logs/day04, review section 03 |
| 5 | Information carriers | notes/day05, ai_logs/day05, review section 04 |
| 6 | Optimal-space vector representation | notes/day06, ai_logs/day06, review section 05 |
| 7 | Online prefix-free encoding | notes/day07, ai_logs/day07, review section 06 |
| 8 | Related work 与 bibliography | notes/day08, ai_logs/day08, review section 07 |
| 9 | Correctness 与 complexity 审计 | notes/day09, ai_logs/day09, review section 08 |
| 10 | 最终整合与项目完成 | review sections 09--10, PDFs, README |
