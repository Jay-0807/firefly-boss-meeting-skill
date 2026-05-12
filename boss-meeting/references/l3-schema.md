# L3 长期记忆 文件家族 schema

> L3 = 跨会话、跨日的持久化记忆。boss-meeting 写、boss-recall 读。
>
> **核心 5 件套**（机器友好、严格契约）：people-graph / topic-threads / decision-ledger / action-items / meeting-index。互相用 `日期 + 主标题` 做交叉引用。
>
> **辅助 2 件**（老板友好、独立维护）：
> - `boss_profile.md` —— 老板偏好持久化，**用户可手动编辑**（见第 6 节）
> - `boss-dashboard.md` —— 老板视图，**自动派生、整体重写**（见第 7 节）

## 文件位置（runtime-aware 自适应）

按下面顺序自检：

1. 调用方传 `boss_state_dir` 参数 → 用它
2. `~/.claude/boss/` 存在 → 用它（**Claude Code** 默认）
3. `~/.hermes/boss/` 存在 → 用它（**Hermes Agent** 默认）
4. 都不存在 → 按当前 runtime 创建：Claude Code 创 `~/.claude/boss/`；Hermes 创 `~/.hermes/boss/`；其他 runtime 创 `~/.boss/`

跨平台路径展开：

| 平台 | `~/.claude/boss/` 实际值 | `~/.hermes/boss/` 实际值 |
|-----|------------------------|------------------------|
| Windows | `C:\Users\<user>\.claude\boss\` | `C:\Users\<user>\.hermes\boss\` |
| macOS / Linux | `/Users/<user>/.claude/boss/` 或 `/home/<user>/.claude/boss/` | 对应 `.hermes` |

L3 目录可被 `boss_state_dir` 参数覆盖整个目录路径（推荐配合云盘同步路径使用）。

## 文件总览（核心 5 件套 + 辅助 2 件）

```
~/.claude/boss/
├── people-graph.md       # 【核心】人物维度
├── topic-threads.md      # 【核心】主题/项目维度（跨多次会议的同一议题线）
├── decision-ledger.md    # 【核心】决策维度（含状态：active / superseded / done）
├── action-items.md       # 【核心】待办闭环（含状态：open / in-progress / done / dropped）
├── meeting-index.md      # 【核心】会议索引（一行一场会议，含日期/标题/参与人/原转写位置）
├── boss_profile.md       # 【辅助】老板偏好持久化（用户可编辑，见第 6 节）
└── boss-dashboard.md     # 【辅助】老板视图（自动派生，整体重写，见第 7 节）
```

## 跨文件引用约定

每个条目都用 **`{YYYY-MM-DD} + {主标题简写}`** 作为跨文件锚点，方便 boss-recall 拼出时间线。

例：`2026-04-21 AI客服Demo验收` 这个锚点同时在 5 个文件里出现：
- `meeting-index.md` 是这一行的"主条目"
- `people-graph.md` 中相关人物的「最近出现」字段引用它
- `decision-ledger.md` 中本次会议产生的决策引用它
- `action-items.md` 中本次会议产生的待办引用它
- `topic-threads.md` 中相关主题的"会议轨迹"列表引用它

---

## 1. people-graph.md（人物维度）

### 文件头

```markdown
# 老板会议·人物图谱

> 自动维护：boss-meeting skill 跑完会议后会更新此文件。
> 手动校对：可以直接编辑此文件，下次 skill 完全信任你的版本。

---
```

### 单人物条目

```markdown
## {规范姓名}

- **别名**：{大名 / 小名 / 英文名 / 拼音 / 转写常见错别字}（顿号分隔）
- **角色**：{身份 / 公司 / 与老板的关系}
- **画像**（≤200 字）：{决策风格 + 当前业务焦点 + 性格触发器 + 与老板的协作模式}
- **首次出现**：{YYYY-MM-DD}（{会议主标题简写}）
- **最近出现**：{YYYY-MM-DD}（{会议主标题简写}）
- **历史关键观点**：
  - {YYYY-MM-DD} {一句话观点摘要}
  - {YYYY-MM-DD} {一句话观点摘要}

---
```

### 关键约束

- 画像 **≤200 字硬上限**——超了就重写不堆叠
- 历史关键观点 **≤10 条**——超了就把最早 3 条合并到画像里再删
- 别名是名字归一的核心字段
- "一旦定下规范姓名就不要改"——只在别名加新发现的称呼

---

## 2. topic-threads.md（主题/项目维度）

### 文件头

```markdown
# 老板会议·主题线索

> 跨会议的同一议题持续追踪：上市规划、AI客服、供应链、品牌建设…
> 每个 ## section 是一个主题；同一主题随多场会议持续追加。

---
```

### 单主题条目

```markdown
## {主题名}

- **状态**：active / dormant / archived
- **首次出现**：{YYYY-MM-DD}（{会议主标题简写}）
- **最近活跃**：{YYYY-MM-DD}（{会议主标题简写}）
- **核心相关人**：{人 1}、{人 2}、{人 3}（最多 5 人）
- **当前路径**：{50–150 字摘要——这条主题目前进展到哪里、下一步要做什么}
- **会议轨迹**：
  - {YYYY-MM-DD}（{会议主标题简写}）{一句话该次会议的关键进展}
  - {YYYY-MM-DD}（{会议主标题简写}）{一句话}
  - …
- **关键决策锚**：
  - 见 `decision-ledger.md` 中 `{YYYY-MM-DD} #决策ID` 三条
- **未闭环待办**：
  - 见 `action-items.md` 中 `{YYYY-MM-DD} #待办ID` 两条

---
```

### 主题命名规则

- 短、稳、可识别。一旦定下一般不改
- 长度 4–10 字
- 例（虚构）：`{产品代号} 验收落地`、`{业务方向} 出海路径`、`{公司名} 并购推进`、`{家庭议题} 心智复盘`

### 状态语义

- `active`：本周/本月还在推进
- `dormant`：超过 1 个月没动过——但没有正式停止
- `archived`：明确已结束/被取代——boss-recall 检索时降权但仍可查到

### 何时新建主题 vs 追加现有主题

新建：会议讨论的核心议题在现有主题里**找不到 ≥60% 重叠**
追加：会议讨论的核心议题与现有主题的"当前路径"逻辑承接

不确定时**追加现有的 + 在「会议轨迹」里加一条**，比硬切两个主题更利于后续追溯。

---

## 3. decision-ledger.md（决策维度）

### 文件头

```markdown
# 老板会议·决策账本

> 每条决策一个 ## section。每条带状态标记，便于 boss-recall 追溯历史拍板。
> ⚠️ 只记真决策（承诺式话术 / 责任+时间齐 / 会议明确收敛后），不记发散讨论。

---
```

### 单决策条目

```markdown
## D-{YYYY-MM-DD-NNN} {决策一句话标题}

- **决策日期**：{YYYY-MM-DD}
- **会议来源**：{会议主标题简写}（见 `meeting-index.md`）
- **决策内容**：{50–120 字。写明决议本身、约束条件、判定标准}
- **责任主体**：{老板姓名 / 团队角色 / 多人}
- **时间窗**：{具体日期 / 时间窗 / 「视 X 而定」}
- **关联主题**：{topic-threads.md 中的主题名}
- **关联人物**：{people-graph.md 中的人 1、人 2}
- **状态**：active / superseded / done / dropped
- **状态变更历史**（可选）：
  - {YYYY-MM-DD} 转为 {新状态}（{原因，如：被 D-XXX 推翻 / 已落地完成}）

---
```

### 决策 ID 规则

`D-{YYYY-MM-DD}-{NNN}`：日期 + 当天序号 3 位。例：`D-2026-04-21-001`、`D-2026-04-21-002`。

ID 一旦分配**永不复用**，即使决策被推翻也保留 ID（仅状态改 superseded）。

### 状态机

```
active  ──┬─→ done       （已落地完成）
          ├─→ superseded  （被新决策推翻——必须在状态变更历史里写明被哪条 D-XXX 替代）
          └─→ dropped     （未落地但明确放弃）
```

**重要**：被 `superseded` 的决策**不删**，是 boss-recall 追溯"我们之前为什么做了这个决定"的关键证据。

### 决策识别启发式（与 boss-meeting/extraction-rules.md 一致）

只把以下三类内容标为决策：

A. **承诺性话术**：「我们决定」「下周开始」「计划在 X 之前完成」
B. **明确责任 + 时间**：具名责任人 + 具体日期/时间窗
C. **会议明确收敛后的结论**：分歧 → 反复辩论 → 一方明确接受

发散讨论、评价性话语、被阻塞议题**不进决策账本**。

---

## 4. action-items.md（待办闭环）

### 文件头

```markdown
# 老板会议·待办账本

> 每条待办一个 ## section。状态滚动更新，便于 boss-recall 追问"上次让我做的事完成没"。
> ⚠️ 三要素必须凑齐：负责人 + 时间 + 动作。缺一标 `(待补)` 不假装齐全。

---
```

### 单待办条目

```markdown
## A-{YYYY-MM-DD-NNN} {待办一句话标题}

- **创建日期**：{YYYY-MM-DD}
- **会议来源**：{会议主标题简写}（见 `meeting-index.md`）
- **动作**：{具体做什么，1–2 行。包含前置条件和判定标准}
- **负责人**：{姓名} 或 {发言人 N（角色）} 或 {(待补)}
- **截止日期**：{YYYY-MM-DD} 或 {本周内} 或 {(待 X 回话后)} 或 {(待补)}
- **关联决策**：{D-XXX-XXX-XXX}（可选，仅当本待办是为某条决策落地）
- **关联主题**：{topic-threads.md 中的主题名}
- **状态**：open / in-progress / done / blocked / dropped
- **状态变更历史**：
  - {YYYY-MM-DD} 创建（来自 {会议}）
  - {YYYY-MM-DD} {状态变更}（{原因}）

---
```

### 待办 ID 规则

`A-{YYYY-MM-DD}-{NNN}`：与决策 ID 同构。

### 状态机

```
open  ─→  in-progress  ─→  done
  │           │
  │           └─→ blocked  ─→ in-progress (待解封)
  │           
  └─→ dropped (明确放弃)
```

`done` 和 `dropped` 的待办在主文件保留 90 天后可归档到 `action-items-archive-{YYYY}.md`，避免主文件无限膨胀。

### 闭环刷新时机

- 老板在新会议中**明确说**"上次的 X 做完了" / "X 还没做" → 更新对应条目状态
- boss-recall 被问到"上次让我做的事"时直接读这个文件，不依赖任何启发式

---

## 5. meeting-index.md（会议索引）

### 文件头

```markdown
# 老板会议·索引

> 一行一场会议。boss-recall 按时间检索的入口文件。

| 日期 | 主标题 | 类型 | 主要参与人 | 决策数 | 待办数 | 转写归档 |
|-----|-------|------|----------|-------|-------|---------|

```

### 单会议条目

每场会议是表格里的一行：

```
| 2026-04-21 | AI客服Demo验收 | 项目验收 | JayJay, 文轩, 西鸣 | 2 | 3 | path/to/transcript.txt |
```

字段说明：

- **日期**：YYYY-MM-DD
- **主标题**：与该次会议的纪要 / 长图主标题保持一致
- **类型**：项目验收 / 战略复盘 / 个人复盘 / 家庭复盘 / 资源对接 / 同业拆解 / 嘉宾分享 / 其他
- **主要参与人**：≤5 人，按发言量排序
- **决策数 / 待办数**：本次会议产生的条目数（链接到 decision-ledger / action-items）
- **转写归档**：原 .txt 转写存放路径——便于 boss-recall 在需要原文时回头取证

### 月度小结追加（推荐）

每月底自动追加一段月度大事记：

```markdown
## 月度大事记 · 2026-04

- **主题级进展**：
  - {主题 A}：{一句话本月进展}
  - {主题 B}：{一句话}
- **关键决策**：D-2026-04-XX-001、D-2026-04-XX-005、D-2026-04-XX-009（合计 3 条核心）
- **未闭环待办**：A-2026-04-XX-002、A-2026-04-XX-007（合计 2 条逾期）
- **新人物入库**：{人 1}、{人 2}
- **图谱回声警告**：{老板某条历史观点本月又一次没用上 → 锁死性下行预警}
```

月度小结是 boss-review skill 的产物，但 boss-meeting 也可以在每月第一次会议时**主动触发一次**回写。

---

## 6. boss_profile.md（老板偏好持久化，用户可编辑）

> 不属于"5 件套核心记忆"——这是**老板偏好配置文件**。第一次跑 boss-meeting 时一次性问完并生成，之后所有 skill 调用从这里读默认值。
>
> 与"5 件套同等待遇"：用户可随时手动编辑，下次 skill 调用完全信任手编版本。

### 文件位置

与 5 件套同目录（runtime-aware）：`~/.claude/boss/boss_profile.md`

### 文件结构

```markdown
---
boss_name: 老王
boss_aliases: [王总, Wang]
default_sharpness: 1
ai_self_reference: AI 参谋
default_team_share: false
preferred_render: local-npm
---

# 老板偏好

> boss-meeting 第一次跑时自动生成，之后默认从这里读。
> 手动编辑此文件可永久覆盖默认值。

## 当前生效

- **老板姓名**：老王
- **别名**（转写中可能的其他写法）：王总 / Wang
- **默认犀利度**：1（可改 1/2/3——见 voice-guide）
- **AI 自称**：AI 参谋（开场气泡用，可改 "外脑" / "助理" / "" 不显示）
- **默认是否生成团队版长图**：否
- **默认渲染建议**：local-npm（写在 Step 8 提示里给老板看）

## 历史变更

- {YYYY-MM-DD} 创建：boss_name=老王、default_sharpness=1
```

### 字段含义

| 字段 | 类型 | 默认值 | 说明 |
|------|-----|--------|------|
| `boss_name` | string | 必填 | 决策长图开场/收尾气泡用的称呼，如 `张总` |
| `boss_aliases` | list | `[]` | 转写中可能的其他写法，用于发言人归一 |
| `default_sharpness` | 1 / 2 / 3 | `1` | voice-guide 的犀利度档位 |
| `ai_self_reference` | string | `"AI 参谋"` | 决策长图开场气泡的自称；空字符串 `""` 表示不出现自称 |
| `default_team_share` | bool | `false` | 是否默认同时输出团队版长图（占位字段，对应 plan P0-3）|
| `preferred_render` | string | `"local-npm"` | 输出后给老板的渲染建议：`local-npm` / `pandoc` / `markdown-only` |

### 何时问、何时不问

- **文件不存在 + 调用方未传 `boss_name`** → 第一次跑 boss-meeting 时**一次性 batch 问** boss_name + boss_aliases + default_sharpness（其他字段用默认值），问完立即写文件
- **文件存在** → **永远不再问**这些字段，所有调用从这里读
- **调用方显式传参**（如 `boss_name=张总`）→ 本场覆盖，**不更新文件**（避免误改长期偏好）
- **老板想改偏好** → 直接编辑 `boss_profile.md`，下次调用即生效

### 与调用参数的优先级（高→低）

1. 本次调用显式传参（如 SKILL.md 输入参数 `boss_name=...`）
2. `boss_profile.md` 字段
3. SKILL.md 内置默认值

### 与 5 件套的差异

| 维度 | 5 件套（people-graph 等） | boss_profile.md |
|------|------------------------|-----------------|
| 维护者 | boss-meeting 自动写 | 第一次自动生成，之后用户编辑 |
| 写入时机 | 每场会议 | 仅首次创建 + 用户手动 |
| 内容性质 | 会议事实记录 | 个人偏好配置 |
| 删除影响 | 丢失会议历史 | 下次跑会议时重新问一次 |

---

## 7. boss-dashboard.md（老板视图，自动派生、整体重写）

> 不属于"5 件套核心记忆"——这是 5 件套的**自动派生视图**。每次 boss-meeting 跑完会议（或老板手动触发刷新）时被**整体重写**。
>
> ⚠️ **不要手编**——下次刷新会全量覆盖。

### 文件位置

与 5 件套同目录（runtime-aware）：`~/.claude/boss/boss-dashboard.md`

### 文件结构（每次自动重生成）

```markdown
# 老板视图

*自动生成于 {YYYY-MM-DD HH:MM}，下次 boss-meeting 跑完会刷新。**不要手编。***

## 🔥 本周决策（最近 7 天，最多 5 条）

- [D-2026-05-08-001] 决策一句话标题（status: active）
- ...

## ⏰ 即将到期 / 已逾期待办（按截止日排序，最多 7 条）

- [A-2026-05-01-001] 动作 — 负责人 — 还有 N 天 / 已逾期 N 天
- ...

## ⚠️ 反复盲点（boss-recall 模式诊断）

- 「{盲点描述}」—— 出现于 D-XXX, D-YYY, D-ZZZ
- ...

> 数据不足时（<10 场会议或 <3 次同类盲点命中）此 section 显示「数据不足，建议先攒 ≥10 场会议再看模式诊断」。

## 🟢 活跃主题（最近 30 天有新进展，最多 5 条）

- 主题 — 当前路径一句话 — 上次会议日期
- ...

## 🔴 沉默主题（>30 天无更新，最多 3 条）

- 主题 — 沉默 N 天 — 最近一次状态
- ...

## 📅 上一场会议摘要

*{2026-05-08 X-1 验收会：{15 字一句话结论，复用纪要末尾的斜体总结}}*
```

### 生成时机

- boss-meeting Step 7.6（写完前 5 个文件后）**整体重写**（见 boss-meeting/SKILL.md）
- 老板手动触发：直接调 boss-meeting 传 `mode=refresh_dashboard`（占位约定，实现时再加）

### 数据来源（计算规则）

| section | 计算方式 |
|---------|---------|
| 本周决策 | `decision-ledger.md` 中 `决策日期 ∈ 最近 7 天` 且 `状态 = active`，按日期倒序，最多 5 条 |
| 即将到期/逾期待办 | `action-items.md` 中 `状态 ∈ {open, in-progress, blocked}`，按截止日升序，逾期优先，最多 7 条；截止日为 `(待补)` 的不上榜 |
| 反复盲点 | 跨条扫描 `people-graph.md` 历史关键观点 + `decision-ledger.md` superseded 链 + `action-items.md` 同主题 dropped；同类型 ≥3 次才上榜 |
| 活跃主题 | `topic-threads.md` 中 `状态 = active` 且 `最近活跃 ≤30 天`，按最近活跃倒序，最多 5 条 |
| 沉默主题 | `topic-threads.md` 中 `状态 = active` 但 `最近活跃 >30 天`，按沉默天数倒序，最多 3 条 |
| 上一场会议摘要 | `meeting-index.md` 最后一行 + 当场会议纪要末尾斜体一句话总结 |

### boss-recall 的特殊用法

当老板的查询**意图模糊到无法定位**到 6 类查询之一（例如"那事儿进展呢"），boss-recall **直接返回 boss-dashboard.md 的内容**作为默认答复，不再 AskUserQuestion 反问候选清单（详见 `boss-recall/SKILL.md` Step 1）。

这是 dashboard 最关键的复用场景——把"老板模糊提问"自动转成"看一眼当前局面"。

### 与 5 件套的差异

| 维度 | 5 件套 | boss-dashboard.md |
|------|--------|------------------|
| 写入方式 | 增量追加 / 字段更新 | 整体重写 |
| 用户编辑 | 信任手编 | **禁止手编**（下次刷新覆盖） |
| 数据性质 | 真相源 | 派生视图 |
| 缺失影响 | 丢失会议历史 | 下次跑会议时重新生成，无损失 |

---

## 检索性能与 token 经济

预计满档容量（1 年累积）：

| 文件 | 大小 |
|------|-----|
| people-graph | ~50 KB |
| topic-threads | ~50 KB |
| decision-ledger | ~40 KB |
| action-items | ~30 KB（含 90 天归档线）|
| meeting-index | ~20 KB |
| **合计** | **~190 KB** |

每次 boss-meeting 调用**不要全量加载**——按下面策略选择性载入：

### 加载策略 A：本次相关条目按需加载（默认）

scan 转写关键词，**只载入命中的条目**：

| 文件 | 命中规则 |
|-----|---------|
| people-graph | 转写中出现的发言人姓名（包含别名）|
| topic-threads | 转写中出现的关键词与主题名 50% 字符重叠或语义近 |
| decision-ledger | 上述命中的主题对应的最近 3 条 active 决策 |
| action-items | 上述命中的主题对应的所有 open / in-progress 待办 |
| meeting-index | 最近 30 天 + 命中主题相关的所有会议 |

### 加载策略 B：boss-recall 调用时

按查询意图加载（详见 boss-recall/SKILL.md）：

- 按人查 → 全量加载 people-graph 中该人条目 + 该人涉及的所有主题/决策/待办
- 按主题查 → topic-threads 该主题 + 关联决策/待办/会议索引
- 按时间查 → meeting-index 命中时段 + 同时段所有决策/待办

---

## 文件初始化

skill 第一次运行时如果 `~/.claude/boss/` 不存在或文件缺失：

1. **先创建目录** `~/.claude/boss/`
2. 对每个缺失的文件**写入仅含文件头的空骨架**（见上面各 section 的"文件头"模板）
3. 不要在初始化时写任何示例数据

## 用户手动编辑

5 个文件都是普通 markdown，可以随时打开编辑：

- 修改人物画像 / 主题状态 / 决策状态 / 待办状态
- 拆分错合并的条目
- 删除被错识别的条目

下次 skill 运行时**完全信任**你的版本，不会回滚。

## 与 boss-recall 的契约

boss-recall skill 的工作完全建立在这 5 个文件的格式约定上。**不要破坏字段名、不要改 ID 格式、不要把 markdown 改成 JSON**——契约一变两个 skill 都得改。
