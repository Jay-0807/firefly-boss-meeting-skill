# boss-skills

> 老板的 AI 外脑——把会议转写变成纪要、决策长图，并跨会议记账。

两个配套的 Claude Code skills，专为「老板会议复盘 + 长期记忆」场景设计：

| Skill | 职责 | 触发时机 |
|-------|------|---------|
| **[boss-meeting](./boss-meeting)** | 输入会议转写 → 生成会议纪要 markdown + 决策长图 markdown，同时维护 5 件套长期记忆 | 跑完一场会议后 |
| **[boss-recall](./boss-recall)** | 任意时刻按主题/人/时间/决策/待办/模式追溯历史 | 老板问"上次跟 X 聊的事进展怎么样" |

## 它解决什么问题

老板每天开 N 场会，传统会议软件把每场会议当成孤岛——第二周就忘了上周决定了啥、谁说了啥、上次让我做的事完成了吗、我是不是又一次犯了同一个盲点。

这套 skill 把每场会议变成**持续记账**：5 个 markdown 文件构成一份**可读、可手编、可搜的长期记忆**。下次会议遇到同一人时自动调取历史背景做更深的诊断；老板任何时候随口一问，boss-recall 直接给出时间线 + 决策链 + 当前状态。

## 安装

### 全局 Claude Code skill

复制 `boss-meeting/` 和 `boss-recall/` 两个目录到：

- Windows: `C:\Users\<your-name>\.claude\skills\`
- macOS / Linux: `~/.claude/skills/`

第一次运行 `boss-meeting` 时会自动在 `~/.claude/boss/` 创建 5 件套核心 L3 文件 + 通过一次性问答生成 `boss_profile.md`（老板偏好持久化）。后续每场会议跑完会另外刷新 `boss-dashboard.md`（老板视图）。

### 通过 hermes agent

在 hermes 配置里引用 skill 名 `boss-meeting` 和 `boss-recall`，传入参数即可。详见各 SKILL.md 的"输入参数约定"段。

## L3 长期记忆（5 件套核心 + 2 件辅助）

L3 目录按 runtime 自适应：

- **Claude Code** → `~/.claude/boss/`
- **Hermes Agent** → `~/.hermes/boss/`
- **其他 LLM 框架** → `~/.boss/`，或调用方通过 `boss_state_dir` 参数自定义（推荐配合云盘同步路径）

```
{boss_state_dir}/
├── people-graph.md      # 【核心】人物维度：每人 ≤200 字画像 + 历史关键观点
├── topic-threads.md     # 【核心】主题/项目维度：跨多次会议的同一议题线
├── decision-ledger.md   # 【核心】决策维度：每条决策含状态(active/superseded/done)
├── action-items.md      # 【核心】待办闭环：跟踪每条 todo 是否完成
├── meeting-index.md     # 【核心】会议索引：一行一场会议
├── boss_profile.md      # 【辅助】老板偏好持久化（用户可手动编辑）
└── boss-dashboard.md    # 【辅助】老板视图（自动派生、整体重写、不要手编）
```

完整字段约定 + runtime 自检逻辑见 [boss-meeting/references/l3-schema.md](./boss-meeting/references/l3-schema.md)。

## 渲染管线（本地优先）

`boss-meeting` 输出纯 markdown，决策长图需要外部工具渲染成 .jpg。

> ### 🚨 数据安全红线：禁止上传第三方在线服务
> 决策长图含真实人名 / 商业决策 / 财务数字 / 转写原文片段——**禁止贴到 readpo.com、md2card 等任何在线 markdown→图片服务**。本 skill 默认输出**永远视为敏感**。

**推荐路径**（三选一，全部本地运行）：

| 选项 | 工具 | 适用 |
|-----|------|------|
| **A. 本地 npm（首选）** | `markdown-to-image` 同款 npm 包 + Puppeteer | 视觉最贴近样本，5 分钟一次性安装 |
| **B. Pandoc 工具链** | Pandoc + 自定义 CSS + Chrome headless | 已有 Pandoc 工具链的人 |
| **C. 直接看 markdown** | 任意 .md 预览器 | 应急 / 不需要图 |

详见 [boss-meeting/references/rendering-pipeline.md](./boss-meeting/references/rendering-pipeline.md) 的安装与渲染脚本。

## 使用流程

### 场景 1：跑一场会议
```
boss-meeting [传入: transcript；boss_name 等元数据可不传（首次跑会自动问一次）]
  ↓ 自动从转写抽日期/地点/类型 + 从上一场继承 + 从 profile 读偏好
  ↓ 仅当无法自动确定时，发起 1 次 batch AskUserQuestion
  ↓ 生成
- 会议纪要 markdown（顶部斜体标注自动猜测的元数据）
- 决策长图 markdown（用本地渲染器渲染——不上传第三方）
- 5 件套核心 L3 增量更新 + boss-dashboard.md 整体刷新
```

### 场景 2：追溯历史
```
boss-recall [传入: query]
  ↓ 检索 L3
- 时间线 + 决策链 + 当前状态 + 引用源
```

## 调节犀利度

决策长图的"犀利视角"section 默认走**等级 1**（直接点出盲点 + 引用证据）。

```
sharpness=1   # 默认，对所有会议适用
sharpness=2   # 升级到「锁死性下行」级强词（同样错误重复出现时）
sharpness=3   # 直接质疑动机/认知模型（老板特别要求"再狠一点"时）
```

## 设计哲学

这不是会议总结器，是**老板的外脑**。核心价值：

1. **第二人称、不留情面**：决策长图直接对老板说话，盲点必须当场点透
2. **跨会议记账**：5 件套 L3 让每次会议不是孤岛，X 周前的观点会在今天的复盘里被回声引用
3. **图谱回声 = 模式识别**：同一个错误反复出现就是「锁死性下行」，外脑必须升级警告

如果你不想要这种调性，**这套 skill 不适合你**——直接用通用的会议总结工具即可。

## 项目状态

- ✅ `boss-meeting` 可上线
- ✅ `boss-recall` 可上线
- ⏳ `boss-review` 周/月度跨会议摘要 —— 待开发

## License

MIT — 见 [LICENSE](./LICENSE)。

## 贡献

PR / issue 都欢迎。优先方向：
- markdown-to-image 的"老板 AI 外脑"专属主题（紫色气泡 / 阶段彩圆 / 品牌底栏）
- `boss-review` skill 的实现
- 多语言支持（目前是中文优化）
