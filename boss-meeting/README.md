# boss-meeting

老板会议三件套生成器：**会议纪要 + 决策长图 + L3 长期记忆**。

## 输入 → 输出

输入会议转写（带说话人 + 时间戳的多行文本）。输出：

1. **会议纪要 markdown**（中性客观，给团队/外部看；顶部斜体标注自动猜测的元数据）
2. **决策长图 markdown**（第二人称、犀利视角，给老板自己复盘——会被**本地**渲染管线转成 .jpg；禁止上传第三方）
3. **L3 长期记忆**：5 件套核心文件增量更新（人物 / 主题 / 决策 / 待办 / 会议索引）+ `boss-dashboard.md` 整体刷新（老板视图）+ 首次跑时生成 `boss_profile.md`（偏好持久化）

## 调用示例

### Claude Code 直接用

```
/boss-meeting

[然后粘贴会议转写]
```

#### 第一次跑

skill 会**一次性 batch** 通过 AskUserQuestion 问你（仅这一次）：
1. 老板姓名 / 称呼 + 别名
2. 默认犀利度（1/2/3）
3. 任何无法从转写头部抽取的元数据（日期 / 地点 / 类型）
4. 任何置信度 <0.95 的发言人——做成**一个表格**让你勾选式回复

回答完毕，skill 把老板姓名 / 别名 / 犀利度等持久化到 `boss_profile.md`。

#### 第二次起

每次跑会议时，skill 自动按下面优先级补全元数据，**绝大多数场景一句话不问**：

1. `boss_profile.md` 取 boss_name / 别名 / 默认犀利度
2. 转写头部抽取日期 / 地点 / 类型
3. 上一场会议（≤7 天 + 同主题 + 同参会人）继承 location / meeting_type
4. 仅当上面都未确定时，发起 1 次 batch AskUserQuestion

生成的纪要顶部用斜体标注所有"自动猜测的元数据"——老板眼睛扫一行就能挑错，错了说"重跑：date=2026-05-09" 即可。

### 在 Hermes Agent 调用

```yaml
skill: boss-meeting
inputs:
  transcript: |
    {老板姓名} 14:02:11
    今天主要想跟大家对一下…
    （…粘贴完整转写…）
  # 以下全部可选：boss_name 优先从 boss_profile.md 读，date/location/type 优先从转写自动抽
  boss_name: {老板姓名}
  date: 2026-04-21
  location: {地点}
  meeting_type: 项目验收
```

## 输入参数完整列表

| 参数 | 必填 | 默认 | 说明 |
|------|-----|------|------|
| `transcript` | ✅ | — | 会议转写 |
| `boss_name` | 可选 | `boss_profile.md` → batch 询问 | 本场会议被外脑对话的老板姓名（持久化在 profile，绝大多数场景不需重传）|
| `date` | 可选 | 转写自动抽取 → today 兜底 | YYYY-MM-DD |
| `location` | 可选 | 转写抽取 → 上一场继承 → 留空 | 会议地点 |
| `meeting_type` | 可选 | 自动归类（置信 ≥0.7） | 项目验收 / 战略复盘 / 个人复盘 / 家庭复盘 / 资源对接 / 同业拆解 / 嘉宾分享 / 其他 |
| `topic_hint` | 可选 | — | 帮助生成主标题 |
| `boss_state_dir` | 可选 | runtime 自检（Claude Code→`~/.claude/boss/`；Hermes→`~/.hermes/boss/`） | L3 文件目录 |
| `transcript_archive_path` | 可选 | `未归档` | 写入 meeting-index.md 第 7 列 |
| `brand_footer_generator` | 可选 | `firefly--boss-meeting-assistance` | 覆盖 footer "Powered by" 标识 |
| `sharpness` | 可选 | `boss_profile.md` 中的 `default_sharpness`，再缺则 1 | 犀利度等级 1/2/3 |

## 文件结构

```
boss-meeting/
├── SKILL.md                       # 入口，含 frontmatter + 完整工作流
├── README.md                      # 本文件
├── templates/
│   ├── minutes.md                 # 纪要模板
│   └── decision-poster.md         # 决策长图模板
├── references/
│   ├── voice-guide.md             # AI 外脑的语调与犀利度规范
│   ├── l3-schema.md               # 5 件套核心 + 2 件辅助 L3 文件的字段约定
│   ├── extraction-rules.md        # 决策提取、盲点识别、名字归一、转写头部元数据自动抽取
│   └── rendering-pipeline.md      # 渲染管线（本地优先，禁止上传第三方）
└── examples/
    └── full-walkthrough.md        # 端到端流程演示（虚构脱敏例子）
```

## 配套 skill

- [`boss-recall`](../boss-recall/) — 任意时刻按主题/人/时间检索 L3，回答老板的追溯问题
