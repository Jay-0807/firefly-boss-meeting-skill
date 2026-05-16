# 渲染管线：本地优先（markdown-to-image 同款）

skill 输出的是纯 markdown，需要外部工具把它渲染成老板会议长图风格的 .jpg。

> ## 🚨 首要原则：本地渲染、禁止上传第三方
>
> 决策长图含**真实人名 / 商业决策 / 财务数字 / 内部矛盾 / 转写原文片段**。把这段 markdown 贴到 `readpo.com`、`md2card`、`mdbox` 等任何**在线 markdown→图片**服务，等于把会议数据出境给第三方。
>
> **本 skill 默认禁止任何在线渲染路径**。下面 3 种渲染方案**全部本地运行**，不联网、不调用第三方 API。

## 推荐方案对比

| 选项 | 输出 | 安装成本 | 视觉精度 | 适用 |
|-----|------|---------|---------|------|
| **A. 本地 npm `markdown-to-image`** | .jpg / .png | 5 分钟（一次性） | ⭐⭐⭐⭐⭐ 最贴近样本 | 首选；跑得起 Node 的老板 |
| **B. Pandoc + HTML 模板 + 浏览器/wkhtmltopdf** | .pdf / .png | 中（要装 Pandoc + Chrome） | ⭐⭐⭐⭐ | 已有 Pandoc 工具链的人 |
| **C. 直接看 markdown** | 无图，纯文本 | 0 | — | 应急 / 老板不在乎视觉 |

---

## 方案 A：本地 npm `markdown-to-image`（首选）

### 一次性安装

```bash
mkdir -p ~/.claude/boss/render
cd ~/.claude/boss/render
npm init -y
npm install markdown-to-image puppeteer
```

> 这与 `readpo.com` 在线版背后是**同一个 npm 包**（`gcui-art/markdown-to-image`），所以视觉效果一致——区别仅在于**所有渲染都在你本机**，markdown 永远不出本地。

### 一次性渲染脚本

新建 `~/.claude/boss/render/render.js`：

```js
// 用法：node render.js <input.md> <output.jpg> [theme]
const { Md2Poster } = require("markdown-to-image");
const puppeteer = require("puppeteer");
const fs = require("fs");

async function render(mdPath, outPath, theme = "SpringGradientWave") {
  const md = fs.readFileSync(mdPath, "utf8");
  const html = await Md2Poster.toHtml(md, { theme });
  const browser = await puppeteer.launch({ headless: "new" });
  const page = await browser.newPage();
  await page.setContent(html, { waitUntil: "networkidle0" });
  await page.screenshot({ path: outPath, fullPage: true });
  await browser.close();
  console.log(`✅ 已生成 ${outPath}（本地，未上传任何数据）`);
}

const [, , mdPath, outPath, theme] = process.argv;
render(mdPath, outPath, theme).catch(console.error);
```

### 调用

skill 输出决策长图 markdown 后，老板把 markdown 存成 `meeting-2026-05-09.md`，然后：

```bash
node ~/.claude/boss/render/render.js meeting-2026-05-09.md meeting-2026-05-09.jpg
```

完成。`.jpg` 在本地，**整个流程不联网**。

### 主题

`SpringGradientWave`（紫色渐变背景，最贴近样本）。其他可选：`purpleHaze` 等内置主题。长期目标见文末"firefly--boss-meeting-assistance 主题"。

---

## 方案 B：Pandoc + HTML 模板 + 浏览器截图

适合**已经有 Pandoc 工具链**的老板（写论文 / 写文档常用）。

### 一次性配置

```bash
# 1. 装 Pandoc + Chrome（或 wkhtmltopdf）
# 2. 备一个紫色卡片风的 CSS 模板，例：
#    ~/.claude/boss/render/poster.css
```

`poster.css` 关键样式：紫色渐变 body + 紫色 blockquote 卡片 + emoji 锚点放大 + 品牌底栏小字。

### 调用

```bash
pandoc meeting-2026-05-09.md \
  -s --css ~/.claude/boss/render/poster.css \
  -o meeting-2026-05-09.html

# 然后用 Chrome headless 截图
google-chrome --headless --screenshot=meeting-2026-05-09.png \
  --window-size=1080,3000 file://$(pwd)/meeting-2026-05-09.html
```

### 优劣

- ✅ 完全本地、无 npm 依赖
- ✅ CSS 可任意改
- ❌ 需要自己调主题，初期工作量大
- ❌ 紫色气泡 / 阶段彩圆 / type label 加粗等都要手写 CSS

---

## 方案 C：应急——直接看 markdown

老板临时没装任何工具时：

- 在 VSCode / Typora / Obsidian / 任意 markdown 预览器里直接看 .md
- 看不到紫色气泡和品牌底栏视觉，但**所有信息都在文字里**——决策、待办、犀利视角等核心内容不丢

skill 输出的 markdown 本身就是**人类可读的最终物**，渲染成图只是为了便于截图分享。如果你不需要分享，直接看 .md 也行。

---

## skill 输出 → 渲染器的语法映射（共通约定）

无论用方案 A / B / C，skill 都按下面"安全 markdown"输出。**3 种渲染方案都识别同一份 markdown**——切换渲染方案不需要改 skill。

| skill 输出（markdown） | 渲染结果（方案 A / B 共通）|
|----------------------|------------------------|
| `# 主标题` | H1 → 顶部大标题 |
| H1 后第一段普通文本 | 副标题（小字、灰色）|
| `> {老板}, ...` 引用块 | 紫色卡片（开场/收尾气泡）|
| `## emoji + section name` | H2 + emoji 视觉锚点 |
| `### 小节` | H3 加粗 |
| `- 列表` / `1. 数字列表` | 标准列表 |
| markdown 表格 | 卡片矩阵（适合 4-card grid 决策块）|
| emoji 🟢🟡🔴 | 彩色圆点（阶段/严重度锚点）|
| `<sub>...</sub>` | 小号灰字（footer 用）|
| `*斜体*` | 斜体（免责声明用）|

## skill 输出端的 markdown 写法约束

| 元素 | 安全语法 | 不安全（避免使用）|
|-----|---------|------------------|
| 主标题 | `# 标题` | `<h1>...</h1>` 内联 HTML |
| 副标题 | H1 后第一段普通文本（不加任何标记）| `<subtitle>`（非标准），`> ...`（会渲染成气泡），`*斜体*`（会变小字）|
| 紫色气泡 | `> {老板}, ...` blockquote | `<div class="bubble">`（依赖 CSS）|
| section 头 | `## 🧠 名` | `<section>`（依赖渲染 HTML 处理）|
| 卡片矩阵 | markdown 表格 | 自定义卡片语法 |
| 阶段圆圈 | emoji 🟢🟡🔴 + bullet | 颜色 token（不通用）|
| 待办 | `### 编号` + `- 字段：值` | `[ ]` checkbox（部分渲染管线不支持中文标签）|
| 免责声明 | `*斜体*` | `<small>`（部分管线吞掉）|
| 生成器标识 | `<sub>...</sub>` | 隐藏注释 |

---

## ⛔ 关于 readpo.com 等在线版（不推荐）

历史版本的本 skill 推荐过把决策长图 markdown 贴到 `readpo.com/zh/poster` 在线生成图片。**当前版本不再推荐**，原因：

1. **数据出境**：决策长图含真实人名、商业决策、财务数字、转写原文片段——一旦贴到第三方在线服务，等于会议数据被外部服务读到、可能落到第三方日志/缓存
2. **不可控**：服务端的留存策略、隐私政策、缓存策略不在老板控制之中
3. **没有必要**：背后的 npm 包 `gcui-art/markdown-to-image` 完全开源，**本地跑视觉一致**——见上面方案 A

**例外**：纯虚构 / 已脱敏 / 公开演讲稿 这种没有敏感信息的 markdown，可以走在线版。但 生成模式 默认输出**永远视为敏感**。

---

## 视觉对齐验证步骤（首次跑方案 A 时）

1. 用 skill 跑一场虚构会议，把决策长图 markdown 存成 `test.md`
2. 跑 `node render.js test.md test.jpg`
3. 对比与样本 .jpg：
   - 主标题字号 / 副标题位置 ✓
   - 紫色气泡是否正确渲染开场/收尾两处引用块 ✓
   - section 标题的 emoji 锚点显示 ✓
   - 待办区的"姓名 + 日期"对齐 ✓
   - 品牌底栏是否完整显示

如果差距 > 30%（紫色气泡 / 品牌底栏不对），说明需要 fork `markdown-to-image` 写自定义主题（见下文长期路径）。

---

## 长期路径：`firefly--boss-meeting-assistance` 主题

基于 `markdown-to-image` 本地 fork 出老板会议专属主题：

- 紫色气泡卡片样式（开场 / 收尾两处 blockquote）
- 阶段彩圆（🟢🟡🔴 转专属圆点 + 阶段卡片框）
- 待办卡片（统一 bg + 头像/日历 emoji + 卡片间距）
- 品牌底栏精确对齐（QR + slogan + 免责声明）
- type label 高亮（自动识别"这不仅是 X，更是 Y"句式加粗显示）

这就是 footer 里 `<sub>Powered by firefly--boss-meeting-assistance</sub>` 标识的最终归属。

---

## Sources

- [gcui-art/markdown-to-image (GitHub)](https://github.com/gcui-art/markdown-to-image) —— 本地渲染的 npm 包
- [gcui-art/markdown-to-image README_CN](https://github.com/gcui-art/markdown-to-image/blob/main/README_CN.md) —— 安装与主题文档
- ⚠️ ReadPo 在线版（**不推荐**，仅作历史参考）：[readpo.com/zh/poster](https://readpo.com/zh/poster)
