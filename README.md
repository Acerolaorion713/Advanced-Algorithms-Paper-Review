# Advanced Algorithms Paper Review

本项目用于完成论文 *Changing Base without Losing Space* 的阅读、理解与 review 写作。

## 论文信息

- Title: *Changing Base without Losing Space*
- Authors: Yevgeniy Dodis, Mihai Pătraşcu, Mikkel Thorup
- Date: April 11, 2010

## 当前项目结构

```text
Advanced-Algorithms-Paper-Review/
├── README.md
├── paper/
│   └── Changing_base_without_losing_space.pdf
├── notes/
│   ├── day01_paper_overview.md
│   ├── day02_succinct_data_structures.md
│   ├── day03_prefix_free_encoding.md
│   └── day04_sole_encoding.md
├── ai_logs/
│   ├── day01_ai_log.md
│   ├── day02_ai_log.md
│   └── day03_ai_log.md
│   └── day04_ai_log.md
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

`review/` 用于保存正式 LaTeX review。`main.tex` 是主入口，`sections/` 中保存各章节内容。

