<div align="center">

<img src="https://img.shields.io/badge/version-2.0-blue.svg" alt="Version 2.0">
<img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License MIT">
<img src="https://img.shields.io/badge/platform-Claude%20%7C%20Codex%20%7C%20Cowork%20%7C%20Agent%20Skills-orange.svg" alt="Platform Support">
<img src="https://img.shields.io/badge/language-中文%20%7C%20English-brightgreen.svg" alt="Language">

<br><br>

# 🧠 PKB Framework

**Personal Knowledge Base · 个人知识库框架**

Turn any folder into a structured, compound-interest-generating personal knowledge base.

将任意文件夹转化为结构化、可复利增长的个人知识库。

*AI-Powered · Obsidian-Native · Convention over Configuration · Zero Dependencies*

</div>

---

<div align="center">

**[English](#english) | [中文](#中文)**

</div>

---

<a name="english"></a>

## English

### What is PKB Framework?

PKB Framework is an **Agent Skill** that gives your note-taking folder intelligent knowledge management capabilities. It is not another note-taking tool — it adds a structured indexing, smart linking, and mindset distillation layer on top of your **existing notes**.

🎯 **Core philosophy**: Your raw notes are never modified. All value-added operations happen in an independent layer.

### Key Features

| Feature | Description |
|---------|-------------|
| 🏗️ **Three-Layer Architecture** | Raw notes → Summary index (wiki) → AI output — clean separation of concerns |
| ⚙️ **Convention over Configuration** | Create a folder in `raw/` = new category. Delete it = remove category. Zero config files |
| 📝 **Auto-Summarization** | Every note gets a structured summary covering "what / why / how" |
| 🔗 **Smart Linking** | Bidirectional links between notes based on semantic relationships, not just keyword matching |
| 🧬 **Adaptive Mindset Distillation** | Distills thinking patterns from summaries (not full text), auto-discovers 3-5 dimensions |
| 🔍 **Built-in Scanning** | All detection via bash commands — zero external dependencies |
| 🌍 **Cross-Platform** | Claude Code · Claude Desktop (Cowork) · OpenAI Codex · any Agent Skills platform |

### Quick Start

#### Prerequisites

- An AI assistant that supports **Agent Skills** (Claude Code, Claude Desktop, OpenAI Codex, etc.)
- A folder with some Markdown notes (or start from scratch)
- Optional: [Obsidian](https://obsidian.md/) with [Dataview](https://github.com/blacksmithgu/obsidian-dataview) plugin for enhanced visualization

#### Installation

**Claude Code (Terminal CLI)**

```bash
mkdir -p ~/.claude/skills/pkb-framework
cp SKILL.md ~/.claude/skills/pkb-framework/
cp -r references/ ~/.claude/skills/pkb-framework/

cd /path/to/your/notes-folder
claude
# Tell Claude: "Initialize this folder as a PKB using the pkb-framework skill"
```

**Claude Desktop (Cowork)**

In a Cowork session with your notes folder attached:

> "Save the pkb-framework skill and initialize this folder as a PKB"

**OpenAI Codex CLI**

```bash
mkdir -p ~/.codex/skills/pkb-framework
cp SKILL.md ~/.codex/skills/pkb-framework/
cp -r references/ ~/.codex/skills/pkb-framework/
cp agents/openai.yaml ~/.codex/skills/pkb-framework/

cd /path/to/your/notes-folder
codex
```

**Other Agent Skills Platforms**

The Agent Skills standard is supported by a growing ecosystem of tools. PKB Framework works on any platform that follows the standard, including:

| Platform | Installation Path | Notes |
|----------|------------------|-------|
| **Claude Code** | `~/.claude/skills/pkb-framework/` | Terminal CLI, full access |
| **Claude Desktop (Cowork)** | Save via `save_skill` or Cowork plugin | Desktop app, folder-based |
| **OpenAI Codex CLI** | `~/.codex/skills/pkb-framework/` | Terminal CLI, sandbox mode |
| **OpenAI Codex Desktop** | Settings → Skills → Add from folder | Desktop app with GUI |
| **Cursor** | `~/.cursor/skills/pkb-framework/` | AI-first code editor |
| **Windsurf** | Settings → Skills → Import | AI-powered IDE |
| **Vercel Agent** | `~/.agents/skills/pkb-framework/` | Serverless agent platform |
| **Continue** | `~/.continue/skills/pkb-framework/` | Open-source AI IDE extension |

For any platform: copy `SKILL.md` and `references/` to the platform's skills directory. The `agents/openai.yaml` file is Codex-specific and safely ignored by other platforms.

#### Directory Structure

After initialization, your folder will be organized as:

```
your-folder/
├── raw/                    # 📚 Source notes
│   ├── notes/              #   Your categories (auto-detected by folder names)
│   ├── projects/           #   Add any folders you like
│   └── outputs/            #   AI-generated outputs
├── wiki/                   # 🏛️ Knowledge layer
│   ├── nodes/              #   Note summaries with YAML frontmatter
│   ├── mindsets/           #   Distilled thinking patterns
│   ├── index.md            #   Auto-generated content index (Dataview)
│   └── logs/               #   Operation logs
├── PKB操作指南.md           #   Quick reference (auto-generated)
└── PKB-dashboard.md        #   Status dashboard (Dataview-powered)
```

#### Daily Usage

| Command | What it does | When to use |
|---------|-------------|-------------|
| `wiki维护` | Scans for new/orphaned/stale notes, creates summaries, builds links, updates index | After writing new notes |
| `lint检查` | Structural + semantic quality check, generates fix list | Periodic maintenance |
| `mind更新` | Distills your thinking patterns from notes | When you want to understand your mental models |
| Any question | Multi-level search (index → summary → raw notes) with mindset-augmented answers | Knowledge retrieval |

### Workflows

```
Write notes → wiki维护 → Auto-summarize + link + index
                  ↓
Maintain → lint检查 → Find issues → Fix
                  ↓
Understand → mind更新 → Distill mindset patterns
                  ↓
Query → Auto-search → Mindset-augmented answer
```

### Architecture

| Layer | Location | Role | Access |
|-------|----------|------|--------|
| Raw | `raw/` | Original notes, source of truth | Read-only |
| Wiki | `wiki/` | Summaries, index, mindsets | AI-maintained |
| Output | `raw/outputs/` | AI-generated answers | Read-write |

### System Requirements

All detection runs via standard Unix tools with cross-platform fallbacks:

| Tool | Linux | macOS | Windows (Git Bash) | Used for |
|------|-------|-------|---------------------|----------|
| `bash` | ✅ | ✅ | ✅ | Shell execution |
| `find` | ✅ | ✅ | ✅ | File traversal |
| `grep` | ✅ | ✅ | ✅ | Pattern matching |
| `stat` | ✅ (GNU) | ✅ (BSD) | ✅ (MSYS2) | Timestamp comparison |
| `comm` | ✅ | ✅ | ✅ (MSYS2) | Set difference |
| `mktemp` | ✅ | ✅ | ✅ (MSYS2) | Temp file creation |
| `sed` | ✅ | ✅ | ✅ | Stream editing |
| `tr` | ✅ | ✅ | ✅ | Character translation |
| `sort` | ✅ | ✅ | ✅ | File sorting |
| `date` | ✅ | ✅ | ✅ | Date formatting |

---

<a name="中文"></a>

## 中文

### 这是什么

PKB Framework 是一套面向 AI 助手的 **Agent Skill** 指令集，让你的笔记文件夹拥有智能化的知识管理能力。它不是又一个笔记工具——它让你的**已有笔记**自动获得结构化索引、智能关联和思维蒸馏。

🎯 **核心理念**：你的原始笔记永远不被修改，所有增值操作都在独立层完成。

### 核心特性

| 特性 | 说明 |
|------|------|
| 🏗️ **三层架构** | 原始笔记 → 摘要索引 → AI 问答产出，各层职责清晰 |
| ⚙️ **约定优于配置** | 在 `raw/` 下新建文件夹即为新分类，删除即移除分类，零配置文件 |
| 📝 **自动摘要** | 每篇笔记自动生成覆盖"是什么/为什么/怎么做"三要素的结构化摘要 |
| 🔗 **智能关联** | 基于语义关系建立笔记间双向链接，非关键词字面匹配 |
| 🧬 **自适应思维蒸馏** | 从摘要蒸馏思维模式（非全文），自动归纳 3-5 个特征维度 |
| 🔍 **内置扫描** | 全部检测通过 bash 命令完成，零外部依赖 |
| 🌍 **跨平台兼容** | Claude Code · Claude Desktop (Cowork) · OpenAI Codex · 任意 Agent Skills 平台 |

### 快速开始

#### 前置条件

- 支持 **Agent Skills** 的 AI 助手（Claude Code、Claude Desktop、OpenAI Codex 等）
- 一个包含 Markdown 笔记的文件夹（也可从零开始）
- 可选：[Obsidian](https://obsidian.md/) + [Dataview](https://github.com/blacksmithgu/obsidian-dataview) 插件以获得增强可视化体验

#### 安装方式

**Claude Code（终端 CLI）**

```bash
mkdir -p ~/.claude/skills/pkb-framework
cp SKILL.md ~/.claude/skills/pkb-framework/
cp -r references/ ~/.claude/skills/pkb-framework/

cd /path/to/your/notes-folder
claude
# 告诉 Claude：请按照 pkb-framework skill 初始化这个文件夹
```

**Claude Desktop（Cowork 模式）**

在 Cowork 会话中挂载你的笔记文件夹后：

> "请保存 pkb-framework skill 并初始化当前文件夹为 PKB 知识库"

**OpenAI Codex CLI**

```bash
mkdir -p ~/.codex/skills/pkb-framework
cp SKILL.md ~/.codex/skills/pkb-framework/
cp -r references/ ~/.codex/skills/pkb-framework/
cp agents/openai.yaml ~/.codex/skills/pkb-framework/

cd /path/to/your/notes-folder
codex
```

**其他 Agent Skills 平台**

Agent Skills 标准正被越来越多的工具生态所支持。PKB Framework 可在任何遵循该标准的平台上运行：

| 平台 | 安装路径 | 说明 |
|------|----------|------|
| **Claude Code** | `~/.claude/skills/pkb-framework/` | 终端 CLI，完整访问权限 |
| **Claude Desktop (Cowork)** | 通过 `save_skill` 或 Cowork 插件保存 | 桌面应用，基于文件夹 |
| **OpenAI Codex CLI** | `~/.codex/skills/pkb-framework/` | 终端 CLI，沙盒模式 |
| **OpenAI Codex Desktop** | 设置 → Skills → 从文件夹添加 | 带 GUI 的桌面应用 |
| **Cursor** | `~/.cursor/skills/pkb-framework/` | AI 优先的代码编辑器 |
| **Windsurf** | 设置 → Skills → 导入 | AI 驱动的 IDE |
| **Vercel Agent** | `~/.agents/skills/pkb-framework/` | 无服务器 agent 平台 |
| **Continue** | `~/.continue/skills/pkb-framework/` | 开源 AI IDE 扩展 |

对于任意平台：将 `SKILL.md` 和 `references/` 复制到该平台的 skills 目录即可。`agents/openai.yaml` 为 Codex 专用，其他平台自动忽略。

#### 目录结构

初始化后你的文件夹将自动组织为：

```
你的文件夹/
├── raw/                    # 📚 知识库原始材料
│   ├── notes/              #   笔记分类（由文件夹名自动识别）
│   ├── projects/           #   任意添加你需要的文件夹
│   └── outputs/            #   AI 输出笔记
├── wiki/                   # 🏛️ 笔记整理层
│   ├── nodes/              #   笔记摘要（含 YAML frontmatter）
│   ├── mindsets/           #   用户思维蒸馏
│   ├── index.md            #   内容索引页（Dataview 自适应）
│   └── logs/               #   运行日志
├── PKB操作指南.md           #   快速参考（自动生成）
└── PKB-dashboard.md        #   状态仪表盘（Dataview 驱动）
```

#### 日常使用

| 输入 | 作用 | 何时使用 |
|------|------|----------|
| `wiki维护` | 扫描新旧/孤立/过时笔记，生成摘要，建立关联，更新索引 | 写完新笔记后 |
| `lint检查` | 结构+语义质量检查，生成修复清单 | 定期维护时 |
| `mind更新` | 从笔记中蒸馏思维特征 | 想了解自己的思维模式时 |
| 任意问题 | 多级下探检索 → 结合 mindsets 生成个性化回答 | 需要回顾或整合知识时 |

### 工作流

```
写新笔记 → wiki维护 → 自动摘要+关联+索引
              ↓
定期维护 → lint检查 → 发现问题 → 确认修复
              ↓
深度理解 → mind更新 → 蒸馏思维模式
              ↓
知识问答 → 自动检索 → 结合mindsets给出个性化回答
```

### 架构详解

| 层级 | 位置 | 职责 | 读写 |
|------|------|------|------|
| Raw 层 | `raw/` | 原始笔记，核心事实来源 | 只读 |
| Wiki 层 | `wiki/` | 摘要索引与思维蒸馏 | AI 维护 |
| Output 层 | `raw/outputs/` | AI 问答产出 | 读写 |

### 系统要求

所有检测通过标准 Unix 工具完成，跨平台兼容：

| 工具 | Linux | macOS | Windows (Git Bash) | 用途 |
|------|-------|-------|---------------------|------|
| `bash` | ✅ | ✅ | ✅ | Shell 执行 |
| `find` | ✅ | ✅ | ✅ | 文件遍历 |
| `grep` | ✅ | ✅ | ✅ | 模式匹配 |
| `stat` | ✅ (GNU) | ✅ (BSD) | ✅ (MSYS2) | 时间戳对比 |
| `comm` | ✅ | ✅ | ✅ (MSYS2) | 集合差运算 |
| `mktemp` | ✅ | ✅ | ✅ (MSYS2) | 临时文件创建 |
| `sed` | ✅ | ✅ | ✅ | 流编辑器 |
| `tr` | ✅ | ✅ | ✅ | 字符翻译 |
| `sort` | ✅ | ✅ | ✅ | 文件排序 |
| `date` | ✅ | ✅ | ✅ | 日期格式化 |

---

<a name="project-structure"></a>

## Project Structure

```
pkb-framework/
├── SKILL.md                  # Core skill instructions (Agent Skills standard)
├── references/
│   └── SKILL-REF.md          # Bash commands, templates, validation rules
├── agents/
│   └── openai.yaml           # Codex platform metadata & tool declarations
├── README.md                 # This file (bilingual)
├── CHANGELOG.md              # Version history
└── LICENSE                   # MIT License
```

## Design Principles

- 🔒 **Raw notes are immutable** — The `raw/` directory is never modified
- ⚙️ **Convention over configuration** — Directory structure is the only configuration
- 📦 **Zero external dependencies** — All detection via standard Unix tooling
- 🔄 **Incremental-first** — Summaries, links, and mindsets all support incremental updates
- 🌍 **Cross-platform** — Windows (Git Bash/WSL), Linux, macOS all supported
- 💰 **Token efficient** — Progressive loading + summary-based distillation

## Frequently Asked Questions

**Q: Can I use this with an existing folder full of notes?**

A: Yes! The `wiki维护` command automatically scans all existing notes and generates summaries. The first run processes notes in batches and reports progress after each batch.

**Q: Do I need Obsidian?**

A: No. Core functionality works independently. Obsidian + Dataview plugin is recommended only if you want enhanced visual index and dashboard views.

**Q: What languages are supported?**

A: Both Chinese and English. Summary rules and filename sanitization are adapted for both.

**Q: How do I reset?**

A: Delete the `wiki/` directory and `PKB操作指南.md` / `PKB-dashboard.md`. `raw/` stays untouched. The next operation auto-triggers re-initialization.

**Q: What's the performance impact?**

A: Progressive loading keeps daily token consumption low. Core layer is ~340 lines; reference layer is loaded on-demand. Mindset distillation uses summaries rather than full text, reducing tokens by ~75-85%.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

### Development Guidelines

1. Bash code in `references/SKILL-REF.md` must pass `bash -n` syntax validation
2. All bash commands must support Linux (GNU), macOS (BSD), and Windows (Git Bash)
3. Cross-platform compatibility: use POSIX-compatible syntax (`[` over `[[`), provide fallback chains for platform-specific commands (`stat`, `mktemp`)
4. SKILL.md should stay under 500 lines; use `§R` references for detailed content
5. Never use `readlink -f` (macOS incompatible); use `cd + pwd` or `realpath` with fallbacks
6. Always prefix `comm` with `LC_ALL=C` for locale-independent sorting

## Version History

| Version | Date | Highlights |
|---------|------|------------|
| **2.0** | 2026-06-15 | Clean-slate release: removed all legacy migrations, fixed all bash bugs, full cross-platform compatibility, bilingual README |
| 1.5 | 2026-06-09 | Cross-platform compatibility: Agent Skills standard, Codex support |
| 1.4 | 2026-05-06 | Product cleanup, Dashboard quick-command cards |
| 1.0–1.3 | 2026-05 | Architecture iterations, convention-over-config, progressive loading |

See [CHANGELOG.md](./CHANGELOG.md) for full history.

## License

MIT © 2026 — Free to use, modify, and share.

---

<div align="center">

**Built for thinkers, by thinkers. 为思考者而建。**

[Report Bug](https://github.com/nickhck/pkb-framework/issues) · [Request Feature](https://github.com/nickhck/pkb-framework/issues) · [Star on GitHub](https://github.com/nickhck/pkb-framework)

</div>
