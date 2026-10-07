# MemeChat · 克隆笔记

> 目标：把用户提供的 SaaS 设计参考图（wordmark **Enblox**）的视觉 DNA，套用到用户自有的 MemeChat 文案与 App 截图上。
> 产物：单文件静态落地页 `memechat-landing.html` + 本地字体/图片资产。

## 源信息

- 原站 URL: **无**。目标是用户提供的设计参考图 `Responsive-SaaS-Website-Template-in-Framer-with-Modern-Design.jpeg`，不是线上可访问的站点，因此无法做网络抓取或线上比对。
- 源码仓库: 无。
- 原作者: 设计稿来自 Framer Marketplace 的 SaaS 模板（参考图内 wordmark 为 "Enblox"）；页面文案与两张 App 截图来自用户提供的 `website.html` / `preview.webp` / `preview-_1_.webp`。
- 许可证: **专有 / NONE**。模板与截图未随附任何开源许可证，属用户自有的产品素材，**不得对外再分发模板原始素材**。
- 致谢要求: 建议成品保留"灵感来源模板"的说明；本仓库仅在 `memechat-landing.html` 顶部 HTML 注释中标注。

## 技术栈

- 单文件静态站 `memechat-landing.html`：HTML + 原生 CSS + 少量原生 JS（约 40 行）。
- 无框架、无构建步骤、无 Node/运行时依赖，双击或任意静态服务器即可打开。
- 字体：自托管 `assets/fonts/*.woff2` + `assets/fonts/fonts.css`（20 个 @font-face，latin + latin-ext）：
  - Plus Jakarta Sans 400/500/600/700/800（标题+正文）
  - Instrument Serif 400 normal / italic（衬线斜体强调）
  - IBM Plex Mono 400/500/700（微标签、按钮、事实条）
  - **无任何 Google Fonts 热链**。
- 图片：`assets/images/app-chat.webp`（"Hi, nerd!" 聊天屏）、`assets/images/app-home.webp`（"Hello, I'm MemeAI" 动作网格）、`assets/images/memechat-mascot.png`（猫咪头像，从聊天截图裁切）。

## 复刻前预判

- **复杂度等级: L1** — 纯静态营销页，无后端、无路由、无复杂状态、无 WebGL/Canvas、无 3D。
- **推荐模式: 内容爆改** — 保留模板的版式/字号/留白/配色体系，整体替换为 MemeChat 内容。
- 可高保真的部分：
  - 版式骨架（居中首屏 → 大标题 about → 三栏特性 → CTA → 全幅渐变带 → 页脚）
  - 字号层级（超大无衬线标题 + 衬线斜体强调 + 等宽微标签）
  - 黑白灰主色 + "唯一强色 = 落日渐变"的色彩策略
  - 首屏柔光（rose/coral/pink/amber 多径向渐变 + blur + mask）
  - 底部全幅落日渐变带（紫→洋红→玫红→橙→琥珀）
  - 圆角胶囊按钮（实心主按钮 + 描边次按钮）
- 需要近似或替代的部分：
  - 参考图中"手持手机"的实拍手部没有素材 → 用 **CSS 手机外框 + 真实 App 截图** 替代，保留倾斜与上下浮动的手持质感。
  - 模板的确切字体未知 → 采用与 `website.html` 同源的三款字体，观感一致。
  - 参考图的具体渐变节点为肉眼采样 → 已在 `RECON/design-dna.json` 固化为色值。
- 不克隆的部分：Framer 运行时、模板的购买/水印信息、任何第三方追踪。
- 主要风险：**无线上站点可比对**，视觉还原以参考图为准；环境中无 Chrome/Chromium，浏览器类脚本（recon/visual-diff/interaction-probe）无法运行，缺少像素级 diff 证据。

## 跑起来

```bash
cd <当前项目目录>
# 单文件静态站
python3 -m http.server 8123
# 浏览器打开 http://127.0.0.1:8123/memechat-landing.html
```

无需安装依赖。

## 改了什么（对照原版）

- **删了追踪脚本**：克隆版**不含任何** GA / gtag / GTM / 像素/埋点。字体全部本地自托管，无 `fonts.googleapis.com` / `fonts.gstatic.com` 请求。
- 文案 100% 替换为 MemeChat（H1「Meme Smarter, Not Harder.」、副文、about、三栏、chat、CTA、页脚）。
- 配色由模板的"白 + 黑 + 单一强调色"改为**从参考图采样的落日色系**（见替换地图）。
- 导航项由模板的 Home/Services/Features/Blog/Pricing 改为 **Features / Chat / RAG / Download**。
- 新增：移动端汉堡菜单、滚动渐显（IntersectionObserver）、`prefers-reduced-motion` 降级、sticky 头部滚动态毛玻璃。
- 保留：超大标题层级、衬线斜体强调、圆角胶囊按钮、三栏特性、全幅渐变带、悬浮手机舞台。

## 原站 vs 克隆站

| 模块 | 原站表现（参考图） | 克隆实现 | 差异 / 取舍 | 证据 |
|---|---|---|---|---|
| 首屏 | 居中衬线斜体 eyebrow + 超大 H1 + 副文 + 描边胶囊 + 手持手机置于暖色柔光上 | eyebrow「Your memes, finally explained.」+「Meme Smarter, Not Harder.」+ 副文 + 实心黑胶囊 + CSS 手机框置于落日柔光上 | 手部实拍 → CSS 手机框替代；主按钮由描边改为实心黑以强化层级 | `RECON/screenshots/clone-1440.png` |
| 导航 | 品牌左 / 居中五项 / 右侧描边胶囊 CTA | 品牌左（猫咪头像+MemeChat）/ 居中四项 / 右侧描边胶囊 | 项数 5→4，内容替换 | clone-1440.png 顶部 |
| 核心动效 | 静图无法确证；推断为轻微悬浮 | 首屏柔光缓慢漂移、手机手持倾斜+上下浮动、滚动渐显、按钮 hover 反色 | 以图片推断，非抓取所得 | HTML `@keyframes aurdrift/handhold/floaty` |
| 内容区块 | 大标题 about（衬线斜体强调）+ 底对齐副段落 + 三栏特性 + 底部全幅落日带 | 同结构：about + 3 栏（VISION/MEMORY/OFFLINE）+ 中间 chat/demo 面板 + CTA 面板 + 全幅落日带 | 新增 chat/demo 区块（承载第二张真实截图），其余同构 | clone-1440.png 中/下部 |
| 移动端 | 参考图为桌面幅面 | ≤900px 收起为汉堡菜单 + 下拉面板；≤620px 单栏、缩小手机、收紧间距 | 参考图无移动稿，为合理推断 | HTML media queries |

## 复刻评分

- 源证据: **2/5** — 目标是一张静态设计图，无线上站点/源码/网络产物；仅能对参考图与 `website.html` 比对。
- 结构保真: **5/5** — 区块顺序、网格结构、层级与参考图一致。
- 视觉保真: **4/5** — 色值/字号/留白高度还原；手部实拍与模板确切字体为近似替代。
- 动效/交互: **4/5** — 柔光/浮动/渐显/hover 均已实现；参考图为静图，无法逐帧核对。
- 响应式: **4/5** — 全站 `clamp()` + 断点 + 移动菜单；缺多视口截图佐证（见验证缺口）。
- 功能完整: **4/5** — 锚点导航、下载外链、mailto 均可用；无表单/无后端（原站亦为营销页）。
- 内容替换: **5/5** — 全量替换为 MemeChat 文案与真实 App 截图，无模板品牌残留于可见内容。
- 法务/部署风险: **3/5** — 模板素材专有、无可随附许可证；纯静态、零追踪、可任意静态托管，但对外分发素材需获授权。
- **总评: 31 / 40** — L1 内容爆改，视觉与结构高度还原，主要扣分来自"无线上站点可比对"与"手部/字体近似"。

## 替换地图（要换什么改哪）

- 文字 → `memechat-landing.html` 正文区块（首屏 `<h1>`、`.about`、`.acols`、`.chatsec`、`.cta-panel`、`footer`）。
- 图片/媒体 → `assets/images/`（`app-chat.webp`、`app-home.webp`、`memechat-mascot.png`）。
- 配色 → `memechat-landing.html` 顶部 `:root` 变量（`--fg / --muted / --line / --chip / --violet / --magenta / --accent / --orange / --amber`）；核心渐变见 `.aur`、`.cta-aur`、`.band`。
- 字体 → `assets/fonts/`（正文默认 **Inter**、标题 **Plus Jakarta Sans**，另自托管衬线/等宽；改 family 时同步 `fonts.css` 与 `:root` 的 `--sans / --display / --serif / --mono`）。
- 下载链接 → 全站 `https://play.google.com/store/apps/details?id=fun.walawe.memechat`；联系邮箱 → 页脚 MORE 列**直接显示** `dani.zakaria@proton.me`（`mailto:`）。
- 设计身份 → `RECON/design-dna.json`（3 层：design_system / design_style / visual_effects）。

## 验证

- [x] 本地跑通：静态单文件，无构建依赖；`od export` 渲染成功、无报错。
- [x] 截图对照：`RECON/screenshots/original-reference.jpeg`（参考图） vs `RECON/screenshots/clone-1440.png`（克隆站 1440px 整页）。
- [x] `audit-clone.mjs` 输出残留审计报告 → `CLONE_AUDIT.md`（保真度硬伤：未发现；无日文残留；克隆文件无追踪脚本）。
- [x] 渲染修复：修正了 ①桌面端误显移动菜单、②`.cta-aur` 被通配规则覆盖导致 CTA 柔光塌陷、③CTA 猫咪动图不显（改为动画作用于包裹层）。
- [ ] `route-crawl.mjs`：**不适用**（单页站点）。
- [ ] `interaction-probe.mjs`：**未运行** — 环境无 Chrome/Chromium，脚本不可用；交互为纯 CSS + 少量 JS，已人工核对源码。
- [ ] `visual-diff.mjs`：**未运行** — 同上，且无线上原站可比对。
- **验证不了的点（如实记）**：
  1. 无线上原站 → 无法做真实路由/交互/网络抓取证据。
  2. 无系统 Chrome → recon / visual-diff / interaction-probe 无法运行，无像素 diff 报告。
  3. 响应式为 CSS 审查 + `clamp()` 推断，**未做多视口截图**（od export 仅出 1440px 桌面幅面）。
  4. 参考图为静图，动效/交互的"原站表现"为推断，非实测。

## 迭代记录（第 2 轮）

- **移除 GitHub**：导航、移动菜单、CTA 按钮、页脚中的全部 GitHub 链接已删除；空出的导航位改为 **RAG**。
- **简化 about 标题**：`Designed to Help You Do More, With Less Wi-Fi.` → **`Do More, With Less.`**（保留衬线斜体强调）。
- **字体**：新增自托管 **Inter**（variable，latin + latin-ext，2 个 woff2），正文/UI 默认族改为 `'Inter','Plus Jakarta Sans', …`；大标题新增 `--display` 变量并使用 **Plus Jakarta Sans**。衬线（Instrument Serif）与等宽（IBM Plex Mono）保留。
- **新增 RAG 区块**：`#rag`「Answers with receipts.」——检索说明 + 三条事实 + 一个"问答 + 引用来源"卡片（knowyourmeme.com / reddit.com/r/memes / your memes · 12）。导航与页脚同步加 RAG 锚点。
- **页脚联系方式**：由 "Contact" 改为直接显示 **dani.zakaria@proton.me**。
- 重新导出 `RECON/screenshots/clone-1440.png`（1440×4647），区块渲染正常、无横向溢出。
