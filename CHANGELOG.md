# PKB Framework 变更日志

版本格式：0.x / 2.x。主版本号变更意味着不兼容的结构性改动，次版本号变更意味着功能增补或优化。

---

## 2.0 — 2026-06-15

### 定位

全新起点版本。核心思路：**面向所有用户的即装即用 skill**，去除历史迁移包袱，清理开发遗留，修复所有已知 bash 代码错误，确保跨平台完美运行。

### 破坏性变更

- **不再兼容 v1.x 遗留状态**：移除 v1.0 迁移功能（§R.10）、旧版 mindsets 多文件迁移、v1.0 遗留版本检测。新用户从此版本开始使用
- **移除 `wiki/backups/` 机制**：包括 backups 目录结构、backups 规则（§R.4.5）、CLAUDE.md 修改前备份要求
- **移除 `proposals` 存档机制**：简化输出存档判定，仅保留 outputs 一种存档路径

### 修复

- **SKILL-REF.md §R.10 截断**：文件末尾 normalize() 函数断裂，v1.0 迁移功能完全不可用。现随 §R.10 一同移除
- **bash 跨平台兼容性**（P0 级别）：
  - `mktemp` 在 macOS 上须提供模板参数，改为 `mktemp /tmp/pkb.XXXXXX 2>/dev/null || mktemp` 兼容写法
  - `realpath` 在 macOS 上不可用，增加 `readlink -f` 回退
  - `[[` bash 扩展语法改为 `case` 语句（POSIX 兼容）
  - `tr -d '"'` 只剥离双引号，改为同时处理双引号和单引号 `tr -d '"'"'"`
  - `comm` 增加 `LC_ALL=C` 前缀确保跨 locale 排序一致性
  - `set -euo pipefail` 添加到关键 pipeline 防止静默失败
- **.tag-index.json / .processed-files.json 规范强化**：明确 `version` 字段必须存在，明确节点值不含 `.md` 扩展名
- **文件名规则补充**：避免 `.md.md` 双扩展名

### 新增

- **agents/openai.yaml 补全**：增加 `date`、`cut`、`basename`、`readlink` CLI 依赖声明
- **平台中性化**：全文 "Claude" 改为 "AI 助手"，"PKB-dashboard" 中 "对 Claude 说" → "对 AI 助手说"

### 移除

- §R.10 v1.0 迁移指南（完整删除）
- §R.4.5 backups 规则（删除，§R.4.6 重编号为 §R.4.5）
- §R.5.5 / §R.6.1 中的 backups 引用
- SKILL.md §0.4 中 v1.0 遗留 / 旧版 mindsets 版本检测行
- SKILL.md §1 步骤 7（原 v1.0 迁移检测，后续步骤重编号）
- SKILL.md 异常处理表中 v1.0 相关条目
- SKILL.md 术语表中 "版本检测" 和 "v1.0 遗留" 术语
- SKILL.md 存档触发判定中的 proposals 行
- README v1.1-v1.4 对比表（过时信息）
- archive/ 目录中所有 v1.x 历史版本文件

### 版本号跳变说明

跳过大段 1.x 版本号直接升至 2.0，标志此版本为一个新的起点：不再承载任何历史迁移逻辑，所有用户均从此版本开始使用，后续版本号延续 2.x 系列。
- **README 方式三**：OpenAI Codex CLI 安装指引
- **README Obsidian 说明框**：明确核心功能不依赖 Obsidian

### 修复

- **frontmatter 标准化**（兼容性）：移除 `version` 非标准字段，description 精简至约 420 字符（低于 Codex 500 字符限制），去掉外层引号
- **§R.0 根目录定位**（关键）：定位逻辑从「硬依赖 CLAUDE.md」改为「raw/ 目录 + (CLAUDE.md 或 wiki/ 目录) 二选一辅助确认」，Codex 无 CLAUDE.md 也能正确找到 PKB 根目录
- **§R.1 临时文件路径**（健壮性）：从硬编码 `/tmp/` 改为 `mktemp` + `trap EXIT` 自动清理，跨平台更安全

### 优化

- **SKILL-REF.md 移入 `references/`**：符合 Agent Skills 标准的渐进式披露约定
- **平台中性化**：全文去掉 "Claude" 特定称呼，改为中性表述（如 PKB-dashboard 中的 "对 Claude 说" → "对 AI 助手说"）
- **初始化步骤 6**：`CLAUDE.md` 生成从强制改为询问，Codex 等非 Claude 平台跳过
- **版本号可视化**：标题行增加 `(v1.5)` 和兼容平台标识

### 结构变更

```
pkb-framework/
├── SKILL.md              # 核心指令 (改动)
├── references/
│   └── SKILL-REF.md      # 参考规范 (从根目录移入 + §R.0 重写)
├── agents/
│   └── openai.yaml       # Codex 元数据 (新建)
├── README.md             # 用户文档 (增加方式三)
└── CHANGELOG.md          # 变更日志 (新增 v1.5)
```

---

## 1.4 — 2026-05-06

### 定位

产品化清理与体验增强版本。核心思路：**移除开发期遗留概念** + **修复自检清单与 bash 命令不一致** + **Dashboard 快捷指令卡片**，让 skill 更干净、更易上手。

### 破坏性变更

- **移除 `pic` 目录概念**：`pic` 是历史版本遗留的硬编码分类，与 v1.0+ 约定优于配置的设计矛盾。所有文档和 bash 命令中不再出现 `pic` 相关逻辑。用户图片资源推荐使用 Obsidian 插件（obsidian-local-images-plus 等）管理，存储在项目外专用文件夹

### 新增

- **Dashboard 快捷指令卡片**：§R.6.2 PKB-dashboard.md 模板顶部新增 `> [!abstract]` Callout，包含四个指令的表格和"新手入门"提示，用户不看操作指南也能上手
- **README 推荐插件节**：新增"推荐的 Obsidian 插件"章节，推荐 obsidian-local-images-plus（含项目外建图提示）、obsidian-web-clipper、clear-unused-images

### 修复

- **步骤 3 自检清单与 §R.4.3 不一致**（硬性 bug）：SKILL.md §1 步骤 3 自检清单写"仅含字母、数字、短横线、下划线、中文"，但 §R.4.3 已允许英文句点、短横线替换中文标点等。修复：自检清单改为引用 §R.4.3
- **分类发现 bash 命令仍检查 `pic`**（硬性 bug）：§R.1 分类自动发现命令中 `[ "$name" = "pic" ] && continue` 残留，与移除 pic 概念矛盾。修复：删除该行
- **缺失摘要检测 bash 命令仍检查 `pic`**（硬性 bug）：§R.1 缺失摘要检测和增量 mtime 命令中 `[ "$cat" = "pic" ] && continue` 残留。修复：删除该行
- **Dashboard Dataview 查询过滤 `pic`**：`WHERE !contains(file.path, "pic")` 残留。修复：删除 pic 过滤条件
- **孤立 node 删除未清理关联数据**（硬性 bug）：§1 步骤 2 删除孤立摘要时未提及清理 `.tag-index.json` 条目和其他 nodes 关联段中的残留链接。修复：明确删除后须执行两项清理
- **旧版 mindsets 迁移未更新 .processed-files.json**（硬性 bug）：§3 旧版 mindsets 迁移步骤缺少更新 `.processed-files.json` 的操作，导致下次 mind更新 重复处理全部文件。修复：迁移步骤新增第 4 步更新 .processed-files.json
- **v1.0 迁移步骤执行时机不明确**（逻辑错误）：步骤 7 排在步骤 6 之后，但 v1.0 遗留场景须在步骤 1 之前执行。修复：步骤 7 标题下新增执行时机说明
- **孤立 node 删除残留关联链接未记录**：§R.7.2 异常处理表缺少"删除孤立 node 后其他 nodes 关联段残留链接"条目。修复：补充该异常及处理方式

### 移除

- **pic 目录自动跳过规则**：§0.2 约定优于配置表格中"名为 `pic` 的目录 | 自动跳过"行删除
- **目录结构树中 `pic/` 行**：§0.1 三层架构目录树中 `pic/ → 图片资源（自动忽略）` 行删除
- **README 文件说明中 archive/ 目录**：archive 是开发期备份，不属于正式发布文件，从 README 移除

---

## 1.3 — 2026-05-06

### 定位

基于 v1.2 真实环境测试的 Bug 修复与鲁棒性增强版本。核心思路：**修复检测命令缺陷** + **v1.0 遗留状态可恢复** + **中文文件名适配**，解决 outputs 被跳过、迁移命令不可靠、文件名规则过窄、新旧结构冲突等实际运行问题。

### 破坏性变更

- 无结构性破坏，v1.2 用户可直接升级

### 新增

- **§0.4 版本检测机制**：首次执行工作流前自动检测 PKB 结构版本（全新 / v1.0遗留 / v1.1+ / 旧版mindsets），结果缓存至对话上下文
- **v1.0 遗留首次运行指导**：wiki维护 步骤 8 新增推荐执行顺序（先迁移 source_path → 再扫描），避免全 WARNING 不可用结果
- **§R.4.6 CLAUDE.md 版本兼容**：检测旧版 CLAUDE.md 与当前 skill 的规则冲突，提醒用户更新
- **§R.10 多策略匹配迁移**：从单一 H1 grep 改为三级匹配（精确标题→模糊文件名→node文件名反推），大幅提升迁移成功率
- **旧版 mindsets 迁移**：mind更新 时自动检测旧版 4 文件格式，整合为单文件格式并询问是否删除旧文件
- **过时摘要检测 60 秒容差**：避免批量创建后 mtime 接近导致误判
- **v1.0 遗留 / 版本检测** 术语

### 修复

- **缺失摘要检测跳过 outputs 目录**（严重）：§R.1 bash 命令中 `{ [ "$cat" = "outputs" ] || ... } && continue` 错误地将 raw/outputs/ 排除在扫描范围外，导致 outputs 下的笔记永远不会被检测为"缺失摘要"。修复：仅跳过 `pic` 和隐藏目录，不跳过 `outputs`（Bug #1）
- **v1.0 迁移 grep -rl 正则注入**（严重）：§R.10 使用 `grep -rl "^# $title"` 匹配标题，但 title 中的 `.`、`()` 等字符是正则元字符，导致匹配结果不可靠。修复：改为 `grep -rFl`（固定字符串匹配）或逐文件读取 H1 后做字符串比较（Bug #2）
- **文件名规则过窄**（中等）：§R.4.3 仅允许字母/数字/中文/短横线/下划线，但真实文件含 `.`（node.js）、`+`（code+obsidian）、`（）`（中文括号）、`！`、`【】` 等字符。修复：增加英文句点允许、中文括号删除、中文标点替换、加号转 `-plus-`（Bug #3）
- **mtime 命令跨平台兼容**（轻微）：增量 mtime 判定命令 `stat -f %Sm` 仅 macOS 可用。修复：优先使用 `stat -c %y`（Linux），回退到 `date -r`（Bug #4）
- **孤立摘要检测输出不友好**（轻微）：原输出完整路径，信息密度低。修复：改为仅输出文件名和标题

### 优化

- **Description 字段触发词扩展**：新增 知识库/笔记整理/个人wiki/摘要/蒸馏 等中文触发词
- **异常处理表扩展**：新增 v1.0 遗留首次扫描、旧版 mindsets 多文件、标题含正则元字符 3 种异常
- **node 创建步骤明确 outputs 分类**：§R.2 步骤 3 注明 `raw/outputs/` 下文件 category 为 `outputs`

---

## 1.2 — 2026-05-05

### 定位

基于 v1.1 全面测试报告（28 项问题）的 Bug 修复版本。核心思路：**分层加载** + **source_path 集合差** + **安全回退**，解决 Token 消耗过大、安装截断、文件名推导不可靠、回退逻辑误判等严重问题。

### 破坏性变更

- **SKILL.md 拆分为核心层 + 参考层**：核心层 ~260 行作为 skill 注册内容，参考层 ~350 行（SKILL-REF.md）按需读取，初始化时复制到 `wiki/templates/SKILL-REF.md`。同时解决 Token 消耗（Token #1）和 Cowork 安装截断（Issue #1）问题
- **缺失摘要检测改为 source_path 集合差**：彻底消除文件名推导，使用 `comm -23` 做 raw 文件路径与 nodes source_path 的集合差（Bug #1）
- **孤立摘要检测回退逻辑安全化**：缺少 source_path 时输出 WARNING，不再推算路径判定为 ORPHAN（Bug #2）
- **增量 mtime 命令包含 outputs 目录**：不再排除 outputs，由 Claude 在比对时判断归属 mindsets 还是 mindsets_outputs（Bug #4）

### 新增

- **§R.0 PKB 根目录定位**：Cowork 模式下自动查找含 CLAUDE.md 和 raw/ 的目录（Issue #3）
- **§R.10 v1.1 迁移指南**：wiki 维护步骤 7 检测缺少 source_path 的 node 并补充（Bug #2 迁移方案）
- **mindsets.md 首次初始化模板**：明确 `processed_count: 0, dimensions: []`（Bug #6）
- **§R.5.5 CLAUDE.md 最小模板**：初始化时如根目录无 CLAUDE.md 则生成（Issue #7）
- **重置机制**：§0.4 新增重置说明（Issue #5）
- **前置检查**：mind 更新前检测无 node 的新增 raw 笔记，提示先 wiki 维护（Issue #4）
- **首次关联扫描分批**：.tag-index.json 不存在时按分类分批（每批 ≤15 篇），避免上下文溢出（Token #2）
- **mind 分批蒸馏策略细化**：单分类超 30 篇时按文件名排序再分批；每批蒸馏后直接增量追加到 mindsets.md（Token #3）
- **.tag-index.json 和 .processed-files.json 增加 version 字段**（Bug #11）
- **嵌套目录递归扫描**：bash 命令使用 `find` 递归扫描子目录中的 .md 文件（Bug #3）

### 修复

- **缺失摘要检测**：从文件名推导改为 source_path 集合差（Bug #1）
- **孤立摘要回退逻辑**：从推算路径判定改为 WARNING 输出（Bug #2）
- **过时摘要检测注释**：修正为"对比 raw 文件 mtime 与 node 文件 mtime"（Bug #5）
- **首次 mindsets.md 模板明确化**：不再说"空文件仅含 frontmatter"，给出完整初始模板（Bug #6）
- **代码块嵌套冲突**：模板中使用 `~~~~` 作为外层代码块分隔符（Bug #7）
- **关联判定决策树冗余**：简化为三分支，移除"仅关键词字面重复？"冗余判断（Bug #8）
- **source_path 回退逻辑重复**：孤立检测和过时检测不再共享回退逻辑（缺少 source_path 统一输出 WARNING）（Bug #9）
- **frontmatter 字段引号**：title、category、summary、source_path 均用引号包裹（Bug #10）
- **增量 mtime 命令排除 outputs**：改为包含 outputs，递归扫描子目录（Bug #4）
- **嵌套目录不扫描**：`for f in "$cat_dir"*.md` 改为 `find "$cat_dir" -name "*.md" -type f`（Bug #3）

### 优化

- **三层架构说明**：§0.1 明确 Output 层存储于 raw/outputs/，属 Raw 层子目录但功能独立（Logic #1）
- **存档触发判定**：从"200字"改为"超过 3 个段落且包含实质性分析"（Logic #2）
- **初始化覆盖保护**：操作指南和 Dashboard 已存在时询问用户是否覆盖（Logic #3）
- **summary 定义明确化**：从"10-30字"改为"10-30个中文字符/5-15个英文单词"（Logic #4）
- **文件名大小写冲突**：§R.4.3 新增跨平台注意，建议英文部分统一小写（Issue #6）
- **Dataview 版本要求**：§0.2 注明需要 Dataview 0.5+（Issue #8）

---

## 1.1 — 2026-05-05

### 定位

基于 v1.0 全面测试报告的 Bug 修复版本。核心思路：**单文件设计** + **可靠映射** + **基于摘要蒸馏**，解决双文件安装缺陷、文件名推导不可靠、大规模蒸馏上下文溢出等严重问题。

### 破坏性变更

- **SKILL-REF.md 合并到 SKILL.md**：从双文件结构回归单文件设计，解决 save_skill 无法保存双文件的问题（Bug #1, #28, #29）。SKILL-REF.md 保留为空引用层
- **node frontmatter 新增 source_path 字段**：记录 raw 文件相对路径，替代不可靠的文件名推导映射（Bug #5, #34）
- **processed_files 从 frontmatter 移至独立文件**：新增 `wiki/nodes/.processed-files.json`，避免 frontmatter 规模膨胀（Bug #22）

### 新增

- **基于 nodes 摘要蒸馏**：mind 更新以 nodes 摘要为主要输入，仅在需要时读取 raw 原文，token 消耗降低约 75-85%（Bug #24）
- **.processed-files.json**：独立存储已处理文件列表，替代 mindsets frontmatter 中的 processed_files 字段
- **增量 mtime 判定 bash 示例**：新增 §R.1 增量判定命令（Bug #23）
- **术语"孤立页面"**：补充定义，与"孤立摘要"区分（Bug #20）

### 修复

- **标题提取逻辑重写**：跳过 YAML frontmatter 后取 H1 标题，回退到文件名（Bug #4）
- **孤立摘要检测改用 frontmatter**：从 category + source_path 字段读取分类和路径，不再从文件名解析（Bug #8, #9）
- **缺失摘要检测补充过滤**：添加 .pkb-ignore 和隐藏目录过滤（Bug #7）
- **分类发现命令 `||` 优先级修正**：改用 `{ [ ] || [ ]; } && continue` 明确语义（Bug #6）
- **`pic` 目录规则统一**：§0.2 改为"名为 `pic` 的目录"，与 bash 精确匹配一致（Bug #2）
- **摘要长度规则明确优先级**：字数限制优先于比例限制——最少 30 字，最多 300 字（Bug #13）
- **首次执行策略明确**：静默创建适用于 node 创建步骤，确认后继续适用于批次切换（Bug #17）
- **问答下探改为扫描 nodes 目录**：不依赖 index.md 的 Dataview 查询代码（Bug #26）
- **过时摘要检测跨平台兼容**：比较 node 文件自身 mtime 而非解析 created 字段，避免 date 命令兼容性问题（Bug #10）
- **增量关联回写优化**：通过 .tag-index.json 获取候选集，非全量扫描（Bug #16）
- **删除 node 时清理 .tag-index.json**：维护时机增加删除操作（Bug #33）
- **日志目录预检**：步骤 8 明确写入前确认目录存在（Bug #18）

### 优化

- **lint 检查分层**：结构性检查用 bash 完成（低 token），语义检查按需执行，并标注结果受主观判断影响（Bug #19, #21）
- **分批蒸馏**：nodes 超 30 篇时按分类分批读取，避免上下文溢出
- **维度调整阈值量化**：从模糊的"新增大量笔记"改为"超过当前已处理笔记的 30%"（Bug #25）

---

## 1.0 — 2026-05-05

### 定位

基于《PKB-skill 深度优化方案》的全面架构重构。核心思路：**约定优于配置**——让目录结构成为唯一的配置源，消除三重冗余，SKILL.md 分层拆分，内置扫描替代外部脚本。

### 破坏性变更

- **约定优于配置**：分类不再需要在 CLAUDE.md 或任何文件中配置，自动从 raw/ 子目录发现。新增分类只需新建文件夹
- **SKILL.md 分层拆分**：单文件 738 行拆分为核心层 SKILL.md（~250 行）+ 参考层 SKILL-REF.md（~350 行），核心层日常加载，参考层按需读取
- **移除 pkb-scan.js 依赖**：6 项检测能力改为 Claude 内置 bash 命令实现，零外部依赖。脚本归档至 archive/v0.3-scripts/
- **mindsets 自适应维度**：从 2 核心文件 + 固定维度改为 1 必需文件 + Claude 自行归纳维度，首次蒸馏时自适应确定 3-5 个维度
- **index.md 自适应查询**：从每分类一段 Dataview 查询改为单一通用查询（支持 GROUP BY 分组），无论分类数量都不需改动

### 新增

- **.tag-index.json**：nodes 关联预筛索引，记录 tags→node 文件名映射，关联扫描范围从 N 篇缩减到 3-8 篇
- **.pkb-ignore**：在 raw/ 子目录中放置空文件即可标记该目录不参与处理
- **忽略目录约定**：`pic` 目录、`.` 开头目录自动跳过，无需配置
- **mindsets processed_files**：frontmatter 中记录已处理文件列表（`文件名|mtime日期`），实现精确增量更新
- **mindsets dimensions**：frontmatter 中记录维度名称列表，保证增量更新时维度稳定
- **批量确认**：wiki 维护扫描结果按类别分组展示，默认提供批量操作选项
- **首次静默创建**：首次 wiki 维护对缺失摘要静默创建，仅异常时暂停
- **环境检查懒加载**：仅在全新 PKB、用户手动触发或异常时执行环境检查

### 优化

- **SKILL.md 瘦身**：从 738 行降至 ~250 行核心层，日常 token 消耗减少约 60%
- **操作指南和 Dashboard 按需生成**：仅首次初始化时生成，不再每日覆盖
- **Dashboard 通用模板**：单一 Dataview 查询替代每分类一段，维护成本归零
- **双路径逻辑移除**：消除"脚本存在→用脚本，脚本不存在→手动检测"的双路径描述（~80 行）
- **消除与 CLAUDE.md 的重复**：SKILL.md 只定义"做什么和怎么做"，模板细节属于 wiki/templates/，规范定义属于 CLAUDE.md

### 移除

- **scripts/ 目录**：pkb-scan.js 归档至 archive/v0.3-scripts/，不再随 skill 分发
- **§0.4 内嵌模板**：操作指南和 Dashboard 模板移至 SKILL-REF.md R.6
- **§5 模板速查**：改为引用表格（§5），全文移至 SKILL-REF.md R.5
- **§6 校验规则汇总**：核心规则内联在步骤中，详解移至 SKILL-REF.md R.4

---

## 0.3 — 2026-05-03

### 定位

基于"PKB 轻量级增强方案 v2"的全面重构。核心思路：不改架构，用最小投入解决运行效率最高、出错最频繁的瓶颈，同时降低操作门槛、提升状态可视性。

### 破坏性变更

- **SKILL.md 从规范视角改为执行视角**：整体结构从"应该是什么样"改为"应该怎么做"，每个工作流包含完整的输入/输出/步骤/自检清单/异常处理
- SKILL.md 章节结构从数字编号（1-8）改为 §0-§7 执行手册格式

### 新增

- **CLI 增量检测脚本 pkb-scan.js**：零依赖 Node.js 脚本（约 400 行），6 项检测能力
- **操作指南页 PKB操作指南.md**：Skill 加载时自动生成/更新
- **Dashboard 页 PKB-dashboard.md**：用 Dataview 查询展示知识库实时状态
- **SKILL.md 执行手册结构**：分 §0-§7 八个章节
- **scripts/ 目录**：初始化时自动创建

### 优化

- **wiki维护工作流重构**：步骤 1 改为调用 pkb-scan.js 获取增量报告
- **lint检查增强**：增加脚本预检步骤
- **创建 nodes 步骤细化**：从模板+质量标准扩展为 11 步完整流程
- **关联判定决策树**：从表格描述改为流程图式
- **文件名合法性校验规则统一**
- **异常处理表**
- **边界规则扩展**

---

## 0.2 — 2026-05-02

### 破坏性变更

- **raw/ 目录结构通用化**：移除固定的 notes/programs/projects 三分类，改为用户自由配置
- **mindsets 架构重构**：从 4 个固定文件改为 2 个核心文件 + 可选扩展文件

### 新增

- 配置项新增「蒸馏维度」选项
- index 模板中的分类段改为动态生成
- nodes 模板中 `category` 字段改为引用用户配置
- 扩展 mindset 机制

### 优化

- 首次执行 wiki 维护时 outputs 固定在最后处理
- mindsets_general.md 整合原 0.1 中三个文件的通用维度
- 初始化流程中的目录树示例改为最简配置

---

## 0.1 — 2026-05-02

### 初始版本

- 三层架构设计：Raw 层 → Wiki 层 → Output 层
- 4 个固定笔记分类：notes/programs/projects/outputs
- 6 种页面类型：logs/backups/outputs/nodes/index/mindsets
- 6 个工作流：基于wiki回答/wiki维护/lint检查/mind更新/logs记录/backups
- Obsidian 特性应用：Dataview/Callout/双向链接
