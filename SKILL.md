---
name: pkb-framework
description: Set up and operate a Personal Knowledge Base with three-layer architecture (raw → wiki → output), convention-over-configuration design, adaptive categories, built-in scanning, and adaptive mindsets distillation. Use when the user mentions wiki维护, lint检查, mind更新, 知识库, 笔记整理, 个人wiki, 摘要, 蒸馏, or asks questions that require searching their notes.
---

# PKB Framework — 个人知识库框架 (v2.0)

> **兼容平台**：Claude Code · Claude Desktop (Cowork) · OpenAI Codex · 及其他支持 Agent Skills 的平台
> **定位**：面向所有用户的即装即用型知识库管理 skill，无需任何前置配置

---

## §0 架构与初始化

### §0.1 三层架构

- **Raw 层**：用户原始笔记，核心事实来源，只读不改
- **Wiki 层**：摘要索引与思维蒸馏，由 AI 助手维护
- **Output 层**：问答与分析产出，按需生成（存储于 `raw/outputs/`，属 Raw 层子目录但功能独立）

```
raw/                → 知识库原始材料（只读不改）
  [任意子目录]/      → 笔记分类（自动识别，名称自定义）
  outputs/          → AI 输出笔记（系统固定）
wiki/               → 知识库整理层（可修改）
  nodes/            → 笔记摘要
    .tag-index.json → 关联预筛索引
    .processed-files.json → mind 增量追踪
  index.md          → 内容索引页（自适应 Dataview）
  mindsets/         → 用户思维蒸馏（自适应维度）
  logs/             → 运行日志
  templates/        → 页面格式模板
    SKILL-REF.md    → 参考规范（初始化时生成）
```

### §0.2 约定优于配置

分类自动发现——扫描 `raw/` 下的直接子目录，按以下规则处理：

| 目录类型 | 处理方式 |
|----------|----------|
| `raw/outputs/` | 系统固定目录，AI 产出 |
| 以 `.` 开头的目录 | 自动跳过（隐藏目录约定） |
| 含 `.pkb-ignore` 文件的目录 | 自动跳过 |
| 其余子目录 | 自动识别为笔记分类 |

> **嵌套目录**：分类识别仅看一级子目录（`raw/[分类]/`），但笔记检测会递归扫描子目录中的 `.md` 文件（bash 使用 `find` 递归），嵌套子目录不建立新分类。

> **Dataview 版本**：本框架使用的 Dataview 查询语法（`TABLE WITHOUT ID`、`FLATTEN`、`GROUP BY`、`dateformat`）需要 Dataview 0.5+ 支持。

### §0.3 文件路径速查

| 内容 | 路径 | 读写 |
|------|------|------|
| 用户笔记 | `raw/[分类]/` | 只读 |
| AI 输出 | `raw/outputs/` | 可读可改 |
| 摘要 | `wiki/nodes/` | 可读可改 |
| 索引 | `wiki/index.md` | 可读可改 |
| 思维蒸馏 | `wiki/mindsets/` | 可读可改 |
| 日志 | `wiki/logs/` | 可读可改 |
| 关联索引 | `wiki/nodes/.tag-index.json` | 可读可改 |
| 增量追踪 | `wiki/nodes/.processed-files.json` | 可读可改 |
| 核心规范 | `CLAUDE.md` | 可读可改 |
| 参考规范 | `wiki/templates/SKILL-REF.md` | 初始化时生成，按需读取 |

### §0.4 版本检测与初始化

本版本（v2.0）假设用户从此版本开始使用 PKB Framework，不再兼容 v1.x 的遗留状态。

**初始化触发条件**（懒加载）：

1. `wiki/nodes/` 为空或不存在（全新 PKB）
2. 用户明确输入"环境检查"或"PKB初始化"
3. 工作流执行中遇到目录缺失等异常时自动触发

初始化步骤：

1. **定位 PKB 根目录**：向上查找含 `raw/` 子目录的目录（详见 §R.0）
2. 确认 `raw/` 和 `wiki/` 目录结构存在，缺失则补全
3. 生成 `wiki/index.md`（模板见 §R.5.2）
4. 生成 `wiki/mindsets/mindsets.md`（模板见 §R.5.4，正文为空，mind更新时填充）
5. 将 `references/SKILL-REF.md` 复制到 `wiki/templates/SKILL-REF.md`（如 skill 安装目录含有该文件；否则跳过）
6. 检测 CLAUDE.md：
   - 如已存在 → 不做修改
   - 如不存在 → 询问用户是否需要生成（Claude Code 项目配置，非必需）
7. 首次初始化时生成 `PKB操作指南.md` 和 `PKB-dashboard.md`（模板见 §R.6）。如已存在则询问用户是否覆盖
8. 向用户报告初始化完成

> 日常操作跳过环境检查，直接执行工作流。

**重置机制**：如需重新开始，删除 `wiki/` 目录和 `PKB操作指南.md`、`PKB-dashboard.md` 即可，`raw/` 保持不变。下次操作会自动触发重新初始化。

---

## §1 wiki维护

### 触发

用户输入 `wiki维护`

### 步骤 1：扫描知识库状态

定位 PKB 根目录后（§R.0），用 bash 命令检测以下 6 项（命令详见 §R.1）：

1. **缺失摘要**：raw 有但 nodes 无（基于 source_path 集合差判定，不依赖文件名推导）。注意：`raw/outputs/` 下的 .md 文件同样纳入检测
2. **孤立摘要**：nodes 有但 raw 无（source_path 对应的文件不存在）
3. **过时摘要**：raw mtime > node mtime
4. **缺失索引**：index 中无对应条目（仅手动索引模式）
5. **质量问题**：frontmatter 缺失/格式错误
6. **关联完整性**：断裂链接/单向强关联

> 如 `wiki/nodes/.tag-index.json` 存在，利用其加速关联扫描。

### 步骤 2：批量展示扫描结果

将所有待处理项按类别分组展示，默认提供批量操作选项：

- **孤立摘要**：列出 node 名、原因 → 全部删除 / 逐条确认 / 保留全部。删除孤立 node 后须：①清理 `.tag-index.json` 中对应条目；②清理其他 nodes 关联段中指向该孤立摘要的 `[[链接]]`
- **过时摘要**：列出 node 名、过期天数 → 全部刷新 / 逐条确认 / 跳过全部
- **缺失摘要**：列出 raw 文件、分类 → 全部创建 / 逐条确认

**首次执行默认策略**：对缺失摘要静默创建（创建 node 步骤无需逐条确认），仅异常时暂停询问。批次切换（从一个分类转到下一个分类）时报告进度并等待用户确认。

### 步骤 3：创建 nodes

对每条缺失摘要，按 §R.2 详细步骤创建 node。

**核心自检清单：**

- frontmatter 7 字段完整（title/link/category/tags/summary/created/source_path），所有字符串值用双引号包裹
- summary 10-30 个中文字符 / 5-15 个英文单词
- 摘要覆盖三要素（是什么/为什么/怎么做）
- 摘要长度：最少 30 字，最多 300 字，在不超过此范围的前提下尽量接近原文 15%-25%；原文不足 100 字可直接引用原文
- 文件名合法（详见 §R.4.3）
- 关联链接格式 `[[笔记标题]]`

### 步骤 4：建立关联

对每个新增 node：

1. 如 `.tag-index.json` 存在，读取并获取 tags 交集的候选 nodes；如不存在，按分类分批读取 nodes（每批 ≤15 篇），每批内做关联判定并回写
2. 仅对候选集做深度关联判定（决策树见 §R.3）
3. 强关联双向添加链接

> **首次关联扫描分批**：.tag-index.json 不存在时，按分类分批读取 nodes，每批不超过 15 篇，避免上下文溢出。

### 步骤 5：增量关联回写

1. 以新增 nodes 的 tags 和 summary 为线索，通过 `.tag-index.json` 获取候选集
2. 仅对候选集的 nodes 检查关联段，补充因新笔记而应建立但尚未建立的关联
3. 若 `.tag-index.json` 不存在，跳过此步骤

### 步骤 6：更新索引

- **.tag-index.json**：将新 node 的 tags 和文件名追加（规范见 §R.8）
- **index.md**：Dataview 模式无需手动更新；手动模式补充缺失条目

### 步骤 7：首次执行规则

- 判定：`wiki/nodes/` 目录为空或仅含 0 个 .md 文件
- 按自动发现的分类顺序逐批处理（outputs 固定在最后），每批建议不超过 15 篇
- 每批创建完成后报告进度，用户确认后继续下一批
- 支持中断恢复：跳过已存在的条目

### 步骤 8：记录日志

按 §R.5.1 格式写入 `wiki/logs/[yyyy-mm-dd]logs.md`。写入前确认 `wiki/logs/` 目录存在。

### 异常处理

| 异常 | 处理 |
|------|------|
| 文件名冲突 | 文件名末尾追加 `-2`、`-3` 等序号 |
| frontmatter 解析失败 | 跳过关联/质量检测，记录到日志 |
| raw 文件编码异常 | 尝试 GBK 解码，仍失败则跳过并报告 |
| 目录不存在 | 自动创建缺失目录后继续 |
| .tag-index.json 损坏 | 忽略索引，全量扫描关联，lint 时提示重建 |

---

## §2 lint检查

### 触发

用户输入 `lint检查`

### 步骤 1：内置扫描

定位 PKB 根目录后，用 bash 命令检测 6 项（同 §1 步骤 1，命令见 §R.1）。

### 步骤 2：结构性检查（bash 完成，低 token 消耗）

| 检查项 | 判定标准 | 实现方式 |
|--------|----------|----------|
| 原始笔记无摘要 | raw 有 .md 但 nodes 无对应 source_path | bash（§R.1） |
| 孤立摘要 | node 有 source_path 但 raw 无对应文件 | bash（§R.1） |
| frontmatter 缺失/格式错误 | 7 字段不完整 | bash grep |
| 关联断裂 | node 关联段 `[[链接]]` 目标无对应 node | bash + grep |
| .tag-index.json 不一致 | tag 映射与实际 nodes 不匹配 | bash |

### 步骤 3：语义检查（需读取 node 内容，按需执行）

| 检查项 | 判定标准 |
|--------|----------|
| 摘要三要素缺失 | "是什么""为什么""怎么做" 任一要素缺失 |
| mindsets 偏离 | mindsets 描述与笔记实际风格明显不符 |
| mindsets 引用失效 | 引用的笔记标题对应 raw 文件不存在 |

> 语义检查结果受 AI 判断影响，建议用户对语义类建议复核后再确认修改。

### 步骤 4：生成建议清单

不直接修改，按 §R.7.1 格式生成清单。

### 步骤 5：确认后执行

向用户展示清单，确认后逐一修改，完成后记录日志。

---

## §3 mind更新

### 触发

用户输入 `mind更新`

### 增量判定

1. `wiki/mindsets/mindsets.md` 为空或无 `last_updated` → 首次，全量蒸馏
2. 有 `last_updated` → 增量更新

增量精确判定：读取 `.processed-files.json` 中的 `mindsets` 字段，仅处理不在列表中或 mtime 变化的文件。命令见 §R.1。

### 前置检查

执行蒸馏前，检查是否有新增 raw 笔记尚未生成对应 node。如有：

1. 列出这些笔记
2. 提示用户先执行 wiki维护
3. 用户选择跳过或先执行 wiki维护

### 蒸馏策略：基于 nodes 摘要蒸馏

首次蒸馏和增量蒸馏均以 nodes 摘要为主要输入，仅在需要深入理解时按需读取 raw 原文。

**分批蒸馏**：当 nodes 超过 30 篇时，按分类分批读取。单分类超 30 篇时按文件名排序再分批（每批 ≤30 篇）。每批蒸馏后直接增量追加到 mindsets.md（而非最后统一汇总），最后做一次全局校验。

### 蒸馏步骤

**3a. mindsets.md — 通用思维特征（唯一必需文件）**

蒸馏范围：raw/ 下所有非 outputs、非忽略目录的笔记对应的 nodes 摘要。

首次蒸馏时，在阅读 nodes 摘要后自行归纳 3-5 个最显著的思维特征维度，用 `> [!insight]` Callout 标注。

启发式问题集（引导归纳，非硬编码维度）：

- 用户关注哪些领域？记录价值的评判标准？
- 用户的认知和信息处理模式？
- 笔记反映的核心价值观和优先级？
- 用户的表达风格和行文偏好？
- 用户使用工具和技术的方式？
- 用户解决问题的思路和路径？

**维度稳定性**：首次蒸馏确定维度后，后续增量更新保持维度名称稳定（记录在 frontmatter `dimensions` 字段中）。仅在维度内容为空或新增笔记数量超过当前已处理笔记的 30% 时调整维度。

**3b. mindsets_outputs.md — 人机协作模式（raw/outputs 有内容时创建）**

蒸馏 raw/outputs/ 下笔记对应的 nodes 摘要，维度自适应。

**3c. 扩展文件（可选）**

当某个分类笔记超过 15 篇且领域特征鲜明时，向用户建议创建 `mindsets_[分类名].md`。

### 输出校验

- frontmatter 包含 `last_updated`、`processed_count`、`dimensions`
- 各维度 Callout 有实质内容
- 非首次执行时在原有基础上增补调整，非全量替换
- 更新 `.processed-files.json`

---

## §4 问答

### 触发

用户提问时自动触发

### 下探顺序

1. **wiki/nodes/ 目录扫描**：列出所有 node 文件名，或读取 `.tag-index.json` 获取 tag 索引
2. **wiki/nodes/ 摘要文件**：读取相关 node 的摘要
3. **raw/ 原始文件**：读取 source_path 或 link 字段指向的原始笔记全文

> 不依赖 index.md 的 Dataview 查询代码做下探。

### mindsets 融合

- mindsets 有内容：结合用户思维特征和偏好生成回答
- 为空：跳过

### 输出与存档

1. 以 `.md` 格式输出到 `raw/outputs/`
2. 命名：`[yyyy-mm-dd] 问题领域-核心结论.md`（2-4字领域 + 4-8字结论，≤50字符，避特殊字符）

### 存档触发判定

| 条件 | 处理 |
|------|------|
| 执行工作流指令、单纯查看/搜索、短对话无新产出 | 不触发 |
| 产出超过 3 个段落且包含实质性分析、用户明确要求保留、多文件交叉分析 | 触发 |

触发后向用户询问确认，确认后存入。

---

## §5 术语表

| 术语 | 含义 |
|------|------|
| 蒸馏 | 从原始笔记中提取用户思维特征和模式，关注"人"而非"内容" |
| 下探 | 从索引→摘要→原文的逐层深入搜寻过程 |
| node | 一篇原始笔记的摘要页，存放在 `wiki/nodes/` 下 |
| mindset | 对用户思维模式和行为特征的提炼总结，存放在 `wiki/mindsets/` 下 |
| 孤立摘要 | `wiki/nodes/` 中存在但 `raw/` 中已无对应原始笔记的 nodes |
| 孤立页面 | wiki/ 下无其他文件链接的 .md 文件（与"孤立摘要"不同） |
| 待更新摘要 | 原始笔记修改时间晚于对应 nodes 创建时间的 nodes |
| 强关联 | 笔记描述同一场景/同一问题的不同阶段，需双向链接 |
| 弱关联 | 共享技术栈但场景不同，单方链接附理由 |
| source_path | node frontmatter 中记录的原始文件相对路径，用于可靠定位 |

---

## §6 参考资源

本 skill 的核心工作流依赖详细的参考规范文件。执行任何工作流步骤时，若遇到 `§R.X` 引用，**必须先读取对应章节获取完整内容，不可凭记忆执行**。

### 资源位置（按优先级）

1. 项目内副本：`wiki/templates/SKILL-REF.md`（首次初始化时生成，随项目走）
2. Skill 安装目录：`references/SKILL-REF.md`（随 skill 分发）

### §R 索引

| 引用 | 章节 | 内容 | 何时读取 |
|------|------|------|----------|
| §R.0 | PKB 根目录定位 | 多平台根目录查找逻辑 | 任何工作流启动时（首先执行） |
| §R.1 | bash 检测命令 | 分类发现、缺失/孤立/过时摘要检测、mtime 增量判定 | wiki维护 §1 步骤 1、lint检查 §2 步骤 1、mind更新 增量判定 |
| §R.2 | node 创建步骤 | 11 步完整创建流程 | wiki维护 §1 步骤 3 |
| §R.3 | 关联判定决策树 | 强/弱/不关联三分支判定 + tag-index 加速机制 | wiki维护 §1 步骤 4 |
| §R.4 | 校验规则详解 | frontmatter 7 字段、摘要三要素、文件名合法性、outputs 命名 | node 创建自检、lint检查 结构检查 |
| §R.5 | 模板全文 | logs/index/nodes/mindsets/CLAUDE.md 完整模板 | 初始化 §0.4、wiki维护 §1 步骤 8、node 创建 |
| §R.6 | 操作指南与 Dashboard 模板 | PKB操作指南.md + PKB-dashboard.md | 首次初始化 §0.4 步骤 7 |
| §R.7 | Lint 建议清单格式 | 结构性问题/语义性问题/mindsets 问题清单模板 + 异常处理详情 | lint检查 §2 步骤 4 |
| §R.8 | .tag-index.json 规范 | JSON 格式、维护时机、设计原则 | wiki维护 §1 步骤 6 |
| §R.9 | .processed-files.json 规范 | JSON 格式、设计说明 | mind更新 增量判定、输出校验 |

### 读取指令

当需要某个 §R 章节的内容时，按以下步骤执行：

1. 读取 `wiki/templates/SKILL-REF.md`（项目内已初始化的副本）
2. 如该文件不存在，回退到 skill 安装目录下的 `references/SKILL-REF.md`
3. 定位到对应章节（以 "## §R.X" 开头）
4. 读取该章节完整内容（从 "## §R.X" 到下一个 "## §R.Y" 或文件末尾）
5. 严格按照读到的命令/模板/规则执行

> **路径说明**：`wiki/templates/SKILL-REF.md` 相对于 PKB 根目录；`references/SKILL-REF.md` 相对于 skill 安装目录。如果你不确定 skill 安装目录的位置，可以通过列出当前已加载的 skill 文件来定位。
