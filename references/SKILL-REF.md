---
description: "PKB Framework reference — bash commands, templates, validation rules. Loaded on-demand by SKILL.md via §R references. Copied to wiki/templates/ on first init."
---

# PKB Framework — 参考规范

> 本文件由 SKILL.md 按需引用。AI 助手执行各工作流步骤时，根据步骤中的 §R 引用读取对应章节。

---

## §R.0 PKB 根目录定位

所有 bash 命令假设工作目录为 PKB 根目录。由于不同平台的工作目录可能不同，需要在每次工作流启动时定位 PKB 根目录。

**多平台定位逻辑**（按优先级尝试）：

```bash
pkb_root="."
found=0

# 方案 A：当前目录就是 PKB 根目录（最常见情况）
if [ -d "$pkb_root/raw" ]; then
  # 任一辅助信号存在即确认（避免将任意有 raw/ 的文件夹误判为 PKB）
  if [ -f "$pkb_root/CLAUDE.md" ] || [ -d "$pkb_root/wiki" ]; then
    found=1
  fi
fi

# 方案 B：向上查找（处理从子目录启动的情况）
if [ "$found" -eq 0 ]; then
  while [ "$pkb_root" != "/" ]; do
    pkb_root="$pkb_root/.."
    # 跨平台路径规范化：使用 cd + pwd 替代 realpath/readlink（macOS readlink 不支持 -f）
    pkb_root=$(cd "$pkb_root" 2>/dev/null && pwd) || break
    if [ -d "$pkb_root/raw" ]; then
      # CLAUDE.md（Claude Code）或 wiki/ 目录（通用信号）二选一满足即可
      if [ -f "$pkb_root/CLAUDE.md" ] || [ -d "$pkb_root/wiki" ]; then
        found=1
        break
      fi
    fi
  done
fi

if [ "$found" -eq 0 ]; then
  echo "ERROR: PKB root not found." >&2
  if [ -d "./raw" ]; then
    echo "REASON: raw/ directory found but no wiki/ or CLAUDE.md present — run PKB initialization first." >&2
  else
    echo "REASON: No raw/ subdirectory found in current or parent directories." >&2
    echo "HINT: Navigate to your PKB folder before starting the agent." >&2
  fi
  exit 1
fi

cd "$pkb_root" || exit 1
echo "PKB root: $(pwd)"
```

> **设计说明**：
> - 「raw/ 目录存在」是唯一必需条件——所有平台都有此目录
> - 「CLAUDE.md 或 wiki/ 存在」是辅助确认信号——两者满足其一即可
> - Claude Code 用户：CLAUDE.md 满足辅助信号
> - Codex / 已初始化项目用户：wiki/ 目录满足辅助信号
> - 完全空白项目（raw/ 存在但 wiki/ 和 CLAUDE.md 都不存在）需初始化后再操作

---

## §R.1 bash 检测命令

以下命令提供检测能力，在 PKB 根目录下执行。所有临时文件使用 `mktemp` 创建并以 `trap` 保证清理。

下文中以 `$PKB_ROOT` 代指 §R.0 定位到的 PKB 根目录路径，实际操作中应先 `cd "$PKB_ROOT"`。

### 分类自动发现

```bash
find raw/ -maxdepth 1 -type d | tail -n +2 | while read d; do
  name=$(basename "$d")
  [ "$name" = "outputs" ] && continue
  case "$name" in .*) continue ;; esac
  [ -f "$d/.pkb-ignore" ] && continue
  echo "$name"
done
```

### 缺失摘要检测（基于 source_path 集合差）

不再通过文件名推导判定缺失，改为 raw 文件路径集合与 nodes 的 source_path 集合做差集：

```bash
set -euo pipefail
# 使用模板参数以兼容 macOS mktemp
raw_list=$(mktemp /tmp/pkb-raw.XXXXXX 2>/dev/null || mktemp)
node_sources=$(mktemp /tmp/pkb-nodes.XXXXXX 2>/dev/null || mktemp)
trap 'rm -f "$raw_list" "$node_sources"' EXIT

(for cat_dir in raw/*/; do
  cat=$(basename "$cat_dir")
  case "$cat" in .*) continue ;; esac
  [ -f "$cat_dir/.pkb-ignore" ] && continue
  find "$cat_dir" -name "*.md" -type f
done) | sort > "$raw_list"

# 列出 nodes 的 source_path（处理双引号和单引号两种包裹方式）
(for f in wiki/nodes/nodes_*.md; do
  [ -f "$f" ] || continue
  sp=$(grep '^source_path:' "$f" | sed 's/source_path: *//' | tr -d '"' | tr -d "'")
  [ -n "$sp" ] && echo "$sp"
done) | sort > "$node_sources"

# 差集 = 缺失摘要（LC_ALL=C 确保跨平台排序一致性）
# comm 在发现差异时返回非零退出码，需要 || true 避免 set -e 提前终止
LC_ALL=C comm -23 "$raw_list" "$node_sources" || true
```

> **重要**：步骤中仅跳过隐藏目录和含 `.pkb-ignore` 的目录，**不跳过 `outputs`**。outputs 下的 .md 文件同样需要生成 nodes。

### 孤立摘要检测

从 frontmatter 读取 source_path，检查对应 raw 文件是否存在：

```bash
for f in wiki/nodes/nodes_*.md; do
  [ -f "$f" ] || continue
  # 同时处理双引号和单引号包裹
  source_path=$(grep '^source_path:' "$f" | sed 's/source_path: *//' | tr -d '"' | tr -d "'")
  if [ -z "$source_path" ]; then
    echo "WARNING: $(basename "$f") missing source_path — cannot determine if orphaned"
    continue
  fi
  [ ! -f "$source_path" ] && echo "ORPHAN: $(basename "$f") (source: $source_path)"
done
```

### 过时摘要检测

对比 raw 文件 mtime 与 node 文件 mtime（非 frontmatter created 字段）：

```bash
for f in wiki/nodes/nodes_*.md; do
  [ -f "$f" ] || continue
  source_path=$(grep '^source_path:' "$f" | sed 's/source_path: *//' | tr -d '"' | tr -d "'")
  if [ -z "$source_path" ]; then
    echo "WARNING: $f missing source_path, cannot check staleness"
    continue
  fi
  if [ -f "$source_path" ]; then
    # Linux stat
    raw_ts=$(stat -c %Y "$source_path" 2>/dev/null)
    # macOS stat 回退
    [ -z "$raw_ts" ] && raw_ts=$(stat -f %m "$source_path" 2>/dev/null)
    node_ts=$(stat -c %Y "$f" 2>/dev/null)
    [ -z "$node_ts" ] && node_ts=$(stat -f %m "$f" 2>/dev/null)
    [ -z "$raw_ts" ] || [ -z "$node_ts" ] && continue
    # 容差 60 秒：避免同一分钟内修改导致误判
    diff=$((raw_ts - node_ts))
    [ "$diff" -gt 60 ] && echo "STALE: $f (raw updated after node, diff=${diff}s)"
  fi
done
```

> **容差说明**：对比加入 60 秒容差，因为 wiki维护 批量创建 nodes 时 raw 文件和 node 文件的 mtime 可能非常接近。

### 增量 mtime 判定（mind 更新用）

列出 raw/ 下所有有效 .md 文件及其修改日期，含 outputs 目录：

```bash
for cat_dir in raw/*/; do
  cat=$(basename "$cat_dir")
  case "$cat" in .*) continue ;; esac
  [ -f "$cat_dir/.pkb-ignore" ] && continue
  find "$cat_dir" -name "*.md" -type f | while read f; do
    # Linux stat → macOS date 回退
    mdate=$(stat -c %y "$f" 2>/dev/null | cut -d' ' -f1)
    [ -z "$mdate" ] && mdate=$(date -r "$f" +%Y-%m-%d 2>/dev/null)
    [ -z "$mdate" ] && mdate="unknown"
    echo "$f|$mdate"
  done
done
```

> 将此命令输出与 `.processed-files.json` 中的记录比对：不在列表中的文件为新增，日期不同的文件为变更。outputs 目录的文件由 `mindsets_outputs.md` 独立处理。

---

## §R.2 node 创建详细步骤

对每条缺失摘要，按以下步骤创建 node：

**输入：** raw 文件路径（如 `raw/notes/docker基础.md`）

**步骤：**

1. 读取 raw 文件全文
2. 提取原文标题：跳过 YAML frontmatter（`---` 到 `---` 之间的内容），取第一个 H1 标题（`# 标题`），若无 H1 则取文件名去除 `.md` 扩展名
3. 确定分类：根据 raw 文件所在的一级子目录名映射到 category 值（`raw/outputs/` 下的文件 category 为 `outputs`）
4. 生成文件名：`nodes_[分类]_[标题转文件名].md`（转文件名规则见 §R.4.3）
5. 检查 `wiki/nodes/` 下是否已存在同名文件，若存在则跳过（中断恢复）
6. 生成摘要（三要素 + 长度规则见 SKILL.md §1 步骤 3 自检清单）
7. 确定 tags：从原文内容中提取 2-5 个核心关键词
8. 生成 summary：一句话核心摘要，10-30 个中文字符 / 5-15 个英文单词
9. 填充 frontmatter（所有字符串值均用双引号包裹）：
   ```yaml
   title: "{{原文标题}}"
   link: "[[原文标题]]"
   category: "{{分类名}}"
   tags: [tag1, tag2]
   summary: "{{一句话摘要}}"
   created: "{{yyyy-mm-dd HH:mm}}"
   source_path: "{{raw/分类名/原文文件名.md}}"
   ```
   > 所有字符串值均用双引号包裹，防止 YAML 特殊字符（冒号、井号、方括号等）导致解析失败。source_path 统一使用双引号，确保 bash 命令可正确剥离。
10. 扫描已有 nodes 建立关联（见 §R.3）
11. 写入文件

---

## §R.3 关联判定决策树

```
两篇笔记是否描述同一场景或同一问题的不同阶段？
├─ 是 → 强关联
│   → 双方关联段互相添加 [[对方标题]] — 理由
└─ 否 → 两篇笔记是否共享技术栈/方法论但应用于不同场景？
    ├─ 是 → 弱关联
    │   → 在更相关的一方添加 [[对方标题]] — 理由
    └─ 否 → 不关联（即使关键词重叠也不建立关联）
```

**关联格式：**

```
- [[笔记标题]] — 关联理由
```

关联依据聚焦于笔记描述的场景、目的与手段的内在联系，而非关键词字面匹配。

**tag-index.json 加速关联：**

新增 node 时，如 `.tag-index.json` 存在，先读取获取与新 node 的 tags 有交集的候选 nodes（通常 3-8 篇），仅对候选集做深度判定。如 `.tag-index.json` 不存在（首次执行），按分类分批读取 nodes，每批不超过 15 篇，每批内做关联判定并回写。

---

## §R.4 校验规则详解

### §R.4.1 nodes frontmatter 必填字段

| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| title | string | 是 | 原文标题，与 H1 或文件名一致，双引号包裹 |
| link | string | 是 | 格式 `[[原文标题]]`，Obsidian 双向链接 |
| category | string | 是 | raw/ 下子目录名（自动发现），双引号包裹 |
| tags | array | 是 | 2-5 个核心关键词 |
| summary | string | 是 | 10-30 个中文字符 / 5-15 个英文单词，双引号包裹 |
| created | string | 是 | 格式 `yyyy-mm-dd HH:mm` |
| source_path | string | 是 | raw 文件相对路径，双引号包裹 |

### §R.4.2 摘要三要素自检

每篇 node 的摘要必须覆盖：

1. **是什么**：核心概念是什么？定义是什么？
2. **为什么**：动机是什么？要解决什么痛点？
3. **怎么做**：关键手段/步骤是什么？

若缺失任一要素，需补全。

### §R.4.3 文件名合法性校验

nodes 文件名规则 `nodes_[分类]_[标题转文件名].md`：

- 允许字符：字母（a-z A-Z）、数字（0-9）、中文、短横线（-）、下划线（_）、英文句点（.）
- 中英文括号类字符（`()（）[]【】{}「」`）→ 删除
- `/\|:*?"<>` → 替换为短横线
- 中英文标点（`！？，。；：、·…—~`）→ 替换为短横线
- 空格 → 替换为短横线
- 加号（`+`）→ 替换为 `-plus-`（常见于技术名称如 `code+obsidian`）
- 连续短横线 → 合并为一个
- 首尾短横线 → 去除
- 长度限制 → 80 字符
- 原始文件名若已有 `.md` 扩展名，转文件名时避免生成 `.md.md` 双扩展名

> **跨平台注意**：不同操作系统对文件名大小写敏感度不同。建议英文部分统一为小写。

> 文件名映射通过 node frontmatter 的 `source_path` 字段保证一致性，不依赖文件名推导。

### §R.4.4 outputs 命名规则

- 格式：`[yyyy-mm-dd] 问题领域-核心结论.md`
- 问题领域：2-4 字概括话题方向
- 核心结论：4-8 字概括关键产出
- 全文件名不超过 50 字符
- 避免特殊字符（括号、引号、斜杠）

### §R.4.5 CLAUDE.md 版本兼容

PKB 根目录下的 `CLAUDE.md` 可能由其他来源生成。本 skill 在初始化时询问用户是否生成，如已存在则不做修改。

当 CLAUDE.md 与 SKILL.md 规则冲突时，**以 SKILL.md 为准**。CLAUDE.md 是项目配置文件，可随 skill 升级而更新。

---

## §R.5 模板全文

### §R.5.1 logs 记录模板

**路径：** `wiki/logs/[yyyy-mm-dd]logs.md`

~~~~markdown
# [yyyy-mm-dd] logs

## HH-mm 操作记录
- 操作类型：wiki维护 / 问答 / lint检查 / mind更新
- 涉及文件：`wiki/nodes/xxx.md` `wiki/index.md`
- 关键变更：新增nodes 3篇，index补充索引2条

> [!warning] 异常记录（如无异常则删除此段）
- 异常描述及处理方式
~~~~

规则：同一天多次操作记录在同一文档中；简约表述，但文件名、位置、参数变更等关键信息必须写明。

### §R.5.2 index 页模板

**路径：** `wiki/index.md`

自适应 Dataview 查询（需要 Dataview 0.5+），无需为每个分类写单独查询段：

~~~~markdown
# PKB 知识库索引

## 状态信息
- 最后更新：{{yyyy-mm-dd}}

## 全部笔记

```dataview
TABLE WITHOUT ID file.link AS "笔记", category AS "分类", summary AS "核心摘要"
FROM "wiki/nodes"
SORT category ASC, file.name ASC
```
~~~~

如用户偏好按分类分组显示：

```dataview
TABLE WITHOUT ID file.link AS "笔记", summary AS "核心摘要"
FROM "wiki/nodes"
FLATTEN category AS cat
GROUP BY cat
SORT file.name ASC
```

**手动回退格式**：使用 Markdown 表格，wiki 维护时同步更新。

### §R.5.3 nodes 页模板

**路径：** `wiki/nodes/nodes_[分类]_[标题转文件名].md`

~~~~markdown
---
title: "{{原文标题}}"
link: "[[原文标题]]"
category: "{{分类名}}"
tags: [{{tag1}}, {{tag2}}]
summary: "{{一句话核心摘要，10-30个中文字符/5-15个英文单词}}"
created: "{{yyyy-mm-dd HH:mm}}"
source_path: "{{raw/分类名/原文文件名.md}}"
---

## 摘要

{{总结原文，覆盖"是什么""为什么""怎么做"三要素；最少30字，最多300字，在不超过此范围的前提下尽量接近原文15%-25%；原文不足100字可直接引用}}

## 关联

- [[相关笔记A]] — 关联理由
~~~~

**Markdown 回退**：`[[链接]]` 替换为 `[链接](../raw/分类/文件名.md)` 标准链接；YAML frontmatter 保留。

### §R.5.4 mindsets 模板

**mindsets.md（通用，唯一必需文件）：**

~~~~markdown
---
last_updated: {{yyyy-mm-dd}}
processed_count: {{已处理的笔记数量}}
dimensions: ["维度1", "维度2", "维度3"]
---

> [!insight] {{维度1名称}}
{{蒸馏内容}}

> [!insight] {{维度2名称}}
{{蒸馏内容}}

> [!insight] {{维度3名称}}
{{蒸馏内容}}
~~~~

> 维度名称和数量由首次蒸馏时自适应归纳，后续保持稳定。Callout 类型根据维度语义选择（insight/pattern/style 等）。processed_files 信息独立存储在 `wiki/nodes/.processed-files.json` 中。

**首次初始化模板**（§0.4 步骤 4）：

```yaml
---
last_updated: ""
processed_count: 0
dimensions: []
---
```

（正文为空，mind更新时首次蒸馏填充内容）

**mindsets_outputs.md（raw/outputs 有内容时创建）：** 格式同上，维度自适应。

**mindsets_[分类名].md（扩展文件，可选）：** 当某分类笔记超过 15 篇且领域特征鲜明时创建。格式同上。

**Markdown 回退**：Callout 替换为 `**📌 维度名**` 加粗标题段落。

### §R.5.5 CLAUDE.md 最小模板

如 PKB 根目录下无 CLAUDE.md 且用户选择生成，使用此最小模板：

~~~~markdown
# PKB Configuration

## Core Rules
- raw/ 目录为只读层，不修改用户原始笔记
- wiki/ 目录由 AI 助手维护，修改前需理解现有结构
- 以 SKILL.md 规则为准，CLAUDE.md 为项目补充配置
~~~~

---

## §R.6 操作指南与 Dashboard 模板

**生成条件**：仅在首次初始化时生成，后续不自动覆盖。如已存在则询问用户是否覆盖。lint检查时检查版本一致性。

### §R.6.1 PKB操作指南.md

~~~~markdown
---
pkb_version: "2.0"
last_updated: "{{yyyy-mm-dd}}"
auto_generated: true
---

# PKB 操作指南

> [!tip] 本页由 PKB Framework 自动生成，无需手动编辑

## 快速操作

| 操作 | 触发词 | 说明 |
|------|--------|------|
| Wiki 维护 | `wiki维护` | 扫描 raw/ 笔记，生成/更新对应 nodes，补充索引，建立关联 |
| Lint 检查 | `lint检查` | 扫描 wiki/ 发现矛盾、过时、孤立、质量低劣的内容 |
| Mind 更新 | `mind更新` | 从笔记摘要中蒸馏用户思维特征，更新 mindsets 文件 |
| 知识问答 | 直接提问 | 扫描 nodes 目录 → 读取摘要 → 读取 raw 原文，结合 mindsets 给出详细回答 |

## 操作流程提示

**写完新笔记后：** 输入 `wiki维护` → 自动检测新增笔记 → 生成摘要 → 建立关联 → 更新索引

**定期维护时：** 输入 `lint检查` → 生成问题清单 → 确认后修复

**想了解自己思维特征：** 输入 `mind更新` → 阅读笔记摘要 → 蒸馏思维模式

## 架构速览

    raw/          → 知识库原始材料（只读不改）
      [分类]/     → 笔记分类（自动识别）
      outputs/    → AI 输出（系统固定）
    wiki/         → 知识库整理层（可修改）
      nodes/      → 笔记摘要
      index.md    → 内容索引
      mindsets/   → 思维蒸馏
      logs/       → 运行日志

## 重要规则

- `raw/` 下用户笔记目录是核心事实来源，只读不改
- `raw/outputs/` 内容可读可修改
- 新增分类只需在 `raw/` 下新建文件夹
- 查看实时状态，请打开 [[PKB-dashboard]]
~~~~

### §R.6.2 PKB-dashboard.md

~~~~markdown
---
pkb_dashboard: true
last_updated: "{{yyyy-mm-dd}}"
auto_generated: true
---

# PKB 仪表盘

> [!tip] 本页由 PKB Framework 自动生成，数据由 Dataview 实时查询

## 快捷指令

> [!abstract] 对 AI 助手说这些话即可触发操作
>
> | 指令 | 作用 | 何时使用 |
> |------|------|----------|
> | `wiki维护` | 扫描新笔记，生成摘要，建立关联 | 写完新笔记后 |
> | `lint检查` | 检查知识库质量，生成修复清单 | 定期维护时 |
> | `mind更新` | 蒸馏思维特征 | 想了解自己的思维模式时 |
> | 直接提问 | 基于知识库检索回答 | 需要回顾或整合知识时 |
>
> **新手入门**：先写几篇笔记 → 输入 `wiki维护` → 完成！

## 各分类笔记概览

```dataview
TABLE WITHOUT ID file.link AS "笔记", category AS "分类", summary AS "核心摘要", dateformat(file.mtime, "yyyy-MM-dd") AS "最后修改"
FROM "wiki/nodes"
SORT category ASC, file.mtime DESC
```

## 最近修改的 raw 笔记

```dataview
TABLE WITHOUT ID file.link AS "笔记", dateformat(file.mtime, "yyyy-MM-dd HH:mm") AS "最后修改"
FROM "raw"
WHERE !contains(file.path, "outputs")
SORT file.mtime DESC
LIMIT 10
```

> 详细操作说明见 [[PKB操作指南]]
~~~~

> 无论用户有多少分类，Dashboard 都不需要改动。

---

## §R.7 建议清单格式与异常处理

### §R.7.1 Lint 建议清单格式

~~~~markdown
# Lint 检查建议清单

## 结构性问题

### nodes 质量问题
- [ ] `wiki/nodes/xxx.md`：summary 为空；category 值不合法
- [ ] `wiki/nodes/yyy.md`：缺少 ## 关联 段落

### 关联完整性
- [ ] `wiki/nodes/A.md` → [[不存在的笔记]]：目标笔记无对应 node
- [ ] `wiki/nodes/A.md` ↔ `wiki/nodes/B.md`：A→B 有链接但 B→A 无反向链接

## 语义性问题

- [ ] `wiki/nodes/xxx.md` 与 `wiki/nodes/yyy.md` 对同一事件的描述矛盾

## mindsets 问题

- [ ] mindsets 中某维度内容与当前 raw 笔记风格明显不符
- [ ] mindsets 引用了已不存在的笔记标题
~~~~

### §R.7.2 异常处理详情

| 异常 | 处理 |
|------|------|
| 文件名冲突（标题不同但转文件名后相同） | 文件名末尾追加 `-2`、`-3` 等序号 |
| frontmatter 解析失败 | 跳过该 node 的关联/质量检测，记录到日志 |
| raw 文件编码异常（非 UTF-8） | 尝试 GBK 解码，仍失败则跳过并报告 |
| 目录不存在 | 自动创建缺失目录后继续 |
| .tag-index.json 损坏 | 忽略索引，全量扫描关联，lint 时提示重建 |
| 删除 node 时 .tag-index.json 未同步 | lint 检查时发现并提示修复；删除 node 后主动清理对应条目 |
| 删除孤立 node 后其他 nodes 关联段残留链接 | 删除 node 后扫描所有 nodes 的 `## 关联` 段，移除指向已删除 node 的 `[[链接]]` 行 |
| bash 命令中 mktemp 在 macOS 上无模板参数失败 | 使用 `mktemp /tmp/pkb.XXXXXX 2>/dev/null \|\| mktemp` 兼容写法 |
| 不同 locale 下中文字符排序不一致导致 comm 差集不准 | 使用 `LC_ALL=C comm -23` 确保排序一致性 |

---

## §R.8 .tag-index.json 规范

**路径：** `wiki/nodes/.tag-index.json`

**格式：**

```json
{
  "version": 1,
  "docker": ["nodes_notes_docker基础", "nodes_notes_docker部署hermes-agent"],
  "obsidian": ["nodes_notes_obsidian-skills", "nodes_notes_obsidian文章内的相互链接"],
  "claude-code": ["nodes_notes_CLAUDE-md强制规范项目行为", "nodes_notes_claude-code使用入门"]
}
```

**维护时机：**

- wiki维护 创建新 node 时更新
- wiki维护 删除孤立 node 时清理对应条目
- lint检查 时校验一致性

**设计原则：**

- `version` 字段标识格式版本，便于未来格式变更时区分
- 只记录 tags → node 文件名（不含 .md 扩展名）的映射
- 不存储摘要或关联理由
- 100 篇 nodes 约 50-80 个 tags，文件大小约 2-5KB

---

## §R.9 .processed-files.json 规范

**路径：** `wiki/nodes/.processed-files.json`

**格式：**

```json
{
  "version": 1,
  "mindsets": {
    "raw/notes/docker基础.md": "2026-05-03",
    "raw/notes/obsidian-skills.md": "2026-05-03",
    "raw/outputs/开发环境搭建方案.md": "2026-05-04"
  },
  "last_mind_update": "2026-05-04"
}
```

**设计说明：**

- `version` 字段标识格式版本，便于未来格式变更时区分
- `mindsets` 记录已处理文件路径及修改日期（`yyyy-mm-dd`，含 outputs）
- `last_mind_update` 记录最后一次 mind 更新日期
- mind 更新时比对当前文件修改日期与此记录，仅处理新增/变更文件
- 文件大小：200 篇约 4-6KB
- 此文件仅由 mind更新 工作流读写，wiki