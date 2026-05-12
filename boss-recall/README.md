# boss-recall

老板会议外脑的"追溯"入口——读 L3，不写。

## 它能回答什么

| 问法 | 类型 |
|-----|------|
| "上次跟 X 聊的事进展怎么样？" | 按人查 |
| "上市规划这条线最近怎么样？" | 按主题查 |
| "上个月开过哪些会？" | 按时间查 |
| "我们之前关于 X 决定了什么？为什么不走 Y？" | 按决策查（含推翻溯源）|
| "上次让我做的 X 完成了吗？" | 按待办查 |
| "我自己有没有反复犯同一个盲点？" | 按模式查（图谱回声扫描）|

## 调用示例

### Claude Code 直接用

老板自然语言提问即可：

```
/boss-recall 上次跟马磊聊的研发撮合现在进展怎么样？
```

skill 自动分类意图、加载相关 L3 子集、组装时间线 + 决策链 + 当前状态。

### 在 Hermes Agent 调用

```yaml
skill: boss-recall
inputs:
  query: 上次跟马磊聊的研发撮合现在进展怎么样？
  intent_hint: 按人      # 可选——加速意图识别
  time_window: 2026-03-01 ~ 2026-04-30   # 可选——仅按时间查时
```

## 输入参数

| 参数 | 必填 | 默认 | 说明 |
|------|-----|------|------|
| `query` | ✅ | — | 老板的自然语言问题 |
| `intent_hint` | 可选 | — | 人 / 主题 / 时间 / 决策 / 待办 / 模式 |
| `boss_state_dir` | 可选 | runtime 自检（Claude Code→`~/.claude/boss/`；Hermes→`~/.hermes/boss/`） | L3 文件目录 |
| `time_window` | 可选 | — | 仅按时间查时使用 |

## 输出格式

强制 4 段式：

```
## "{原问题}"

### 📌 直接答复
{1-3 句直接回答 + 含可验证数字或日期}

### 🕐 时间线 / 会议轨迹 / 决策链
{结构化展示，bullet/表格}

### 📋 当前状态
- 🎯 关联决策：D-XXX 状态：active/...
- 🔄 未闭环待办：A-XXX 截止 YYYY-MM-DD 状态：open/...
- 👥 核心相关人

### 🔗 引用源
- meeting-index.md: ...
- decision-ledger.md: D-XXX
- people-graph.md: ## 姓名
```

详见 [references/answer-format.md](./references/answer-format.md)。

## 文件结构

```
boss-recall/
├── SKILL.md                       # 入口
├── README.md                      # 本文件
├── references/
│   ├── query-patterns.md          # 6 类查询的具体范式与流程
│   └── answer-format.md           # 答复 markdown 的强制结构与例子
└── examples/
    └── sample-queries.md          # 6 类查询的虚构端到端例子
```

## 与 boss-meeting 的契约

`boss-recall` **只读**，`boss-meeting` **只写**。两者共享 `~/.claude/boss/` 目录：

- 5 件套核心 L3 文件 → 按意图选择性加载
- `boss-dashboard.md` → 仅当**老板提问意图模糊**（"那事儿进展呢"等无法定位到 6 类之一）时加载，作为默认答复直接返回
- `boss_profile.md` → boss-recall **不读**（profile 是 boss-meeting 的输入，与会议事实无关）

如果 5 件套全部为空（用户从未跑过 boss-meeting），boss-recall 会直接告诉调用方："还没有任何会议记录，请先跑几次 boss-meeting 攒数据。"

## 配套 skill

- [`boss-meeting`](../boss-meeting/) — 跑会议、生成纪要 + 决策长图、写回 L3
