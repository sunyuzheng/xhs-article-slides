---
name: article-to-slides
description: 将 Superlinear Academy 文章转换为小红书「网页截图风」幻灯片，自动输出 1080×1350px PNG 文件。品牌信息已内置，无需重复配置。
---

# Superlinear Academy → 小红书「网页截图风」幻灯片生成器

将一篇 Superlinear Academy 文章转换为一套专业的小红书竖版幻灯片组。每张幻灯片模拟网页截图外观（带浏览器地址栏、品牌 logo、引流 footer），用 puppeteer-core 批量输出高清 PNG。

## 使用方式

```
/article-to-slides
```

---

## 品牌配置（已内置，无需询问）

以下信息固定使用，**不要向用户提问**：

| 配置项 | 值 |
|--------|----|
| 品牌名 | Superlinear Academy / Superlinear AI |
| 网站 URL | `superlinear.academy` |
| Logo 路径 | `/Users/sunyuzheng/Desktop/superlinear/Superlinear_Academy_Logo_Lockups_2026_V1.0/Main-Superlinear_Academy/1_Primary_左对齐最常用/Black/Web/Primary_Black_NoBgd_RGBx72.png` |
| Address bar | `superlinear.academy{article_path}` |
| Footer 引流文案 | `应用商店搜索 Superlinear Academy · 加入 AI 先行者社区` |
| 深色 header/footer logo | 加 `style="filter:invert(1)"` |
| 浅色 header/footer logo | 正常使用 |

> **Logo 找不到时**：用 `find /Users/sunyuzheng/Desktop/superlinear -name "Primary_Black_NoBgd_RGBx72.png" 2>/dev/null | head -1` 找到最新路径。

---

## Claude 执行指令

当用户调用此 skill 时，按以下步骤执行：

### Step 0 — 收集输入（只问这三项）

向用户询问（对话中已有的就不再问）：

1. **文章内容**：URL 或直接粘贴的文章全文。若为 URL，用 WebFetch 获取正文。
2. **文章路径**：URL 中 domain 之后的部分，例如 `/c/ai-resources/ai-native-orgs`（用于 address bar）。若可从 URL 直接解析则不问。
3. **输出目录**：PNG 保存位置。默认为 `~/Desktop/slides-png/`，如用户未指定则直接使用默认值，无需询问。

品牌 logo、网站 URL、引流文案均已内置，**不要询问**。

---

### Step 1 — 规划幻灯片结构

将文章内容切分为以下标准类型（根据内容取舍，一般 10–16 张）：

| 类型 | 说明 |
|------|------|
| **封面 (Cover)** | 深色背景，大字标题 + 副标题 + 核心论点引用框 + 来源声明绿点 |
| **关于/背景** | 可选。Superlinear Academy / Superlinear AI 企业培训背景，可嵌入照片 |
| **TLDR** | 4–6 条核心观点，左黑边框卡片列表 |
| **数据/悖论** | 统计数字卡片网格 + 结论框 |
| **问题/瓶颈** | 每个瓶颈一张（label + 标题 + 正文 + 数据卡 + quote） |
| **案例** | 每个案例一张（绿点来源标注 + 标题 + 逻辑框 + 角色列表） |
| **原则/框架** | 每个原则一张（大背景数字 + 标题 + 正文 + 组件） |
| **行动建议** | 编号列表，每条含标题 + 说明 |
| **结语 (Closing)** | 浅灰背景，大字金句（下划线强调）+ 引流 CTA 黑框。CTA 标题固定用「Superlinear Academy，AI 先行者的赛博风水宝地」，正文引导在 App Store / Google Play 搜索下载，**不贴外部链接**（小红书限流）|

**内容原则**：
- 正文是主角，不要过度使用「强调块」
- 强调方式：`font-weight: 700-900` > 下划线 > 左边黑线框。**禁止黑底白字色块**
- 每张幻灯片内容适量，宁可留白也不要堆砌

---

### Step 2 — 生成 HTML 文件

将 `slides.html` 写入输出目录的父目录（或用户指定位置），使用以下完整设计系统：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>{文章标题} — Slides</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&family=Noto+Sans+SC:wght@300;400;500;700;900&display=swap" rel="stylesheet">
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  background: #C4C4C4;
  padding: 48px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 48px;
  font-family: 'PingFang SC', 'Noto Sans SC', 'Inter', -apple-system, sans-serif;
}
.slide-label {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  letter-spacing: 2px;
  color: #888;
  text-transform: uppercase;
  align-self: flex-start;
  margin-left: calc(50% - 540px);
}

/* ── Slide: 1080×1350px (小红书 4:5) ── */
.slide {
  width: 1080px;
  height: 1350px;
  overflow: hidden;
  border-radius: 16px;
  box-shadow: 0 32px 80px rgba(0,0,0,0.28), 0 4px 16px rgba(0,0,0,0.12);
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
}

/* ── Browser Chrome (macOS) ── */
.chrome { background: #2C2C2E; padding: 13px 20px 11px; display: flex; flex-direction: column; gap: 9px; flex-shrink: 0; }
.chrome.dk { background: #1A1A1C; }
.dots { display: flex; gap: 7px; align-items: center; }
.dot { width: 12px; height: 12px; border-radius: 50%; }
.dr { background: #FF5F56; } .dy { background: #FFBD2E; } .dg { background: #27C93F; }
.urlbar { background: #3C3C3E; border-radius: 7px; padding: 6px 14px; display: flex; align-items: center; gap: 8px; color: #E0E0E5; font-size: 12px; font-family: 'Inter', sans-serif; }
.urlbar .lk { font-size: 11px; opacity: 0.55; }

/* ── Site Header ── */
.sh { background: #fff; border-bottom: 1px solid #EAEAEA; padding: 14px 36px; display: flex; align-items: center; justify-content: space-between; flex-shrink: 0; height: 84px; }
.sh.dk { background: #0F0F0F; border-bottom: 1px solid #1E1E1E; }
.logo { height: 56px; object-fit: contain; }
.bc { font-size: 11px; color: #BBBBBB; font-family: 'Inter', sans-serif; }
.bc b { color: #555; font-weight: 600; }
.sh.dk .bc b { color: #999; }

/* ── Site Footer ── */
.sf { background: #F4F4F4; border-top: 1px solid #E8E8E8; padding: 13px 36px; display: flex; align-items: center; justify-content: space-between; flex-shrink: 0; height: 68px; }
.sf.dk { background: #090909; border-top: 1px solid #1A1A1A; }
.sfurl { font-size: 12px; color: #999; font-family: 'Inter', sans-serif; }
.sf.dk .sfurl { color: #888; }
.sflogo { height: 36px; object-fit: contain; opacity: 0.65; }
.sf.dk .sflogo { filter: invert(1); opacity: 0.3; }

.content { flex: 1; overflow: hidden; }

/* ── Shared pad ── */
.pad { padding: 44px 60px 36px; height: 100%; display: flex; flex-direction: column; background: #fff; }
.ptag { font-size: 10px; font-family: 'Inter', sans-serif; font-weight: 700; letter-spacing: 3px; text-transform: uppercase; color: #AAAAAA; margin-bottom: 14px; display: flex; align-items: center; gap: 10px; }
.ptag::before { content: ''; display: block; width: 18px; height: 1.5px; background: #CCC; }
.ph1 { font-size: 42px; font-weight: 900; line-height: 1.18; color: #050505; letter-spacing: -1px; margin-bottom: 28px; }
.ph1 sub { font-size: 20px; color: #AAA; font-weight: 400; display: block; letter-spacing: 0; vertical-align: baseline; margin-bottom: 6px; }

/* ── Body text (主角) ── */
.pbody { font-size: 21px; line-height: 1.85; color: #333; }
.pbody p + p { margin-top: 18px; }
.pbody strong { font-weight: 700; color: #0A0A0A; }
.dv { width: 100%; height: 1px; background: #EAEAEA; margin: 20px 0; }

/* ── Logic box: 左边黑线 + 浅灰底 ── */
.lbox { background: #F6F6F6; border-left: 4px solid #000; border-radius: 0 10px 10px 0; padding: 22px 28px; margin-bottom: 16px; }
.lbox .lbl { font-size: 10px; font-family: 'Inter', sans-serif; letter-spacing: 2px; text-transform: uppercase; color: #AAAAAA; margin-bottom: 8px; }
.lbox .txt { font-size: 21px; line-height: 1.75; color: #1A1A1A; }
.lbox .txt strong { color: #000; font-weight: 800; }

/* ── Quote box ── */
.qbox { border-left: 4px solid #000; padding: 18px 24px; background: #F7F7F7; border-radius: 0 8px 8px 0; margin-top: 18px; }
.qt { font-size: 21px; line-height: 1.72; color: #1A1A1A; font-style: italic; }
.qs { font-size: 12px; color: #AAA; margin-top: 8px; font-style: normal; font-family: 'Inter', sans-serif; }

/* ── TLDR rows ── */
.tlist { display: flex; flex-direction: column; gap: 12px; margin-top: 4px; }
.trow { display: flex; align-items: stretch; background: #F6F6F6; border-radius: 8px; overflow: hidden; border-left: 3.5px solid #000; }
.trow-inner { padding: 17px 22px; font-size: 21px; line-height: 1.65; color: #1A1A1A; }
.trow-inner strong { font-weight: 700; color: #000; }

/* ── Stat grid ── */
.sgrid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 16px; }
.sc { border-radius: 12px; padding: 28px 26px; background: #F4F4F4; }
.sc.blk { background: #000; }
.sc .sn { font-size: 58px; font-weight: 900; font-family: 'Inter', sans-serif; letter-spacing: -3px; line-height: 1; color: #000; margin-bottom: 8px; }
.sc.blk .sn { color: #fff; }
.sc .sl { font-size: 16px; color: #777; line-height: 1.55; }
.sc.blk .sl { color: rgba(255,255,255,0.8); }
.sc .ss { font-size: 11px; color: #CCC; font-family: 'Inter', sans-serif; margin-top: 5px; }
.sc.blk .ss { color: rgba(255,255,255,0.22); }

/* ── Paradox/conclusion box ── */
.pdx { background: #F0F0F0; border-radius: 12px; padding: 26px 34px; text-align: center; }
.pdx-q { font-size: 26px; font-weight: 800; color: #000; line-height: 1.38; }
.pdx-note { font-size: 13px; color: #999; margin-top: 10px; font-style: italic; line-height: 1.5; }

/* ── Data card (bottleneck / research) ── */
.bcard { border-radius: 12px; padding: 22px 28px; background: #F5F5F5; margin-bottom: 14px; }
.bn { font-size: 10px; font-family: 'Inter', sans-serif; letter-spacing: 2.5px; text-transform: uppercase; font-weight: 700; color: #BBB; margin-bottom: 8px; }
.bt { font-size: 20px; font-weight: 800; color: #000; margin-bottom: 10px; }
.bb { font-size: 20px; line-height: 1.78; color: #444; }
.bb strong { color: #111; font-weight: 600; }

/* ── Case study ── */
.case-origin { display: flex; align-items: center; gap: 10px; background: #F0F0F0; border-radius: 8px; padding: 12px 18px; margin-bottom: 18px; }
.co-dot { width: 8px; height: 8px; background: #27C93F; border-radius: 50%; flex-shrink: 0; }
.co-text { font-size: 13px; color: #555; font-family: 'Inter', sans-serif; }
.co-text b { color: #222; font-weight: 700; }
.case-eyebrow { font-size: 10px; font-family: 'Inter', sans-serif; font-weight: 700; letter-spacing: 3px; text-transform: uppercase; color: #BBB; margin-bottom: 8px; }
.case-h { font-size: 36px; font-weight: 900; color: #000; line-height: 1.18; letter-spacing: -0.8px; margin-bottom: 10px; }
.case-badge { display: inline-block; font-size: 12px; font-family: 'Inter', sans-serif; font-weight: 600; color: #888; background: #EBEBEB; padding: 4px 12px; border-radius: 20px; margin-bottom: 20px; }

/* ── Role / trait list ── */
.rlist { display: flex; flex-direction: column; gap: 10px; }
.ri { display: grid; grid-template-columns: 150px 1fr; gap: 16px; background: #F5F5F5; border-radius: 8px; padding: 15px 18px; align-items: start; }
.ri-name { font-size: 13px; font-family: 'Inter', sans-serif; font-weight: 800; color: #000; padding-top: 1px; }
.ri-desc { font-size: 18px; line-height: 1.65; color: #444; }

/* ── Principle slide ── */
.p-bg-num { font-size: 96px; font-weight: 900; font-family: 'Inter', sans-serif; color: #EBEBEB; line-height: 1; letter-spacing: -5px; margin-bottom: -18px; user-select: none; }
.trait-rows { display: flex; flex-direction: column; gap: 10px; margin-top: 2px; }
.tr { display: grid; grid-template-columns: 170px 1fr; gap: 14px; background: #F5F5F5; border-radius: 9px; padding: 15px 18px; }
.trn { font-size: 13px; font-family: 'Inter', sans-serif; font-weight: 800; color: #000; padding-top: 1px; }
.trd { font-size: 15.5px; line-height: 1.62; color: #555; }

/* ── Framework layers ── */
.ctx-rows { display: flex; flex-direction: column; gap: 13px; margin-top: 4px; }
.cr { border-radius: 10px; padding: 22px 26px; background: #F5F5F5; display: flex; gap: 18px; }
.cr-n { font-size: 30px; font-weight: 900; font-family: 'Inter', sans-serif; color: #D8D8D8; line-height: 1; width: 34px; flex-shrink: 0; }
.cr-t { font-size: 17px; font-weight: 700; color: #000; margin-bottom: 5px; }
.cr-b { font-size: 18px; line-height: 1.68; color: #444; }

/* ── Action steps ── */
.arows { display: flex; flex-direction: column; gap: 15px; margin-top: 4px; }
.ar { display: flex; gap: 18px; align-items: flex-start; }
.anum { width: 38px; height: 38px; background: #000; color: #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 16px; font-weight: 800; font-family: 'Inter', sans-serif; flex-shrink: 0; margin-top: 2px; }
.at { font-size: 20px; font-weight: 700; color: #000; margin-bottom: 5px; }
.ab { font-size: 17px; line-height: 1.72; color: #444; }

/* ── Cover slide ── */
.s-cover .content { background: #080808; padding: 56px 72px 48px; display: flex; flex-direction: column; }
.cover-ey { font-size: 11px; font-family: 'Inter', sans-serif; letter-spacing: 3.5px; text-transform: uppercase; color: #666; }
.cover-main { flex: 1; display: flex; flex-direction: column; justify-content: center; }
.cover-title { font-size: 66px; font-weight: 900; line-height: 1.08; color: #fff; letter-spacing: -2px; margin-bottom: 20px; }
.cover-sub { font-size: 24px; color: #888; font-weight: 400; margin-bottom: 44px; line-height: 1.45; }
.cover-box { border-left: 3px solid #fff; padding: 24px 30px; background: rgba(255,255,255,0.04); border-radius: 0 10px 10px 0; margin-bottom: 32px; }
.cover-bt { font-size: 22px; line-height: 1.75; color: #CCC; }
.cover-bt strong { color: #fff; font-weight: 700; }
.cover-source { display: flex; align-items: center; gap: 14px; padding: 14px 18px; background: rgba(255,255,255,0.03); border: 1px solid #1E1E1E; border-radius: 8px; }
.cover-source-dot { width: 6px; height: 6px; background: #27C93F; border-radius: 50%; flex-shrink: 0; }
.cover-source-text { font-size: 13px; color: #888; font-family: 'Inter', sans-serif; line-height: 1.5; }
.cover-source-text b { color: #AAA; font-weight: 600; }
.cover-meta { font-size: 11px; font-family: 'Inter', sans-serif; color: #555; letter-spacing: 2px; text-transform: uppercase; margin-top: 28px; }

/* ── Closing slide ── */
.s-close .content { background: #F8F8F8; padding: 64px 72px; display: flex; flex-direction: column; justify-content: center; }
.cq { font-size: 44px; font-weight: 900; line-height: 1.26; color: #000; margin-bottom: 44px; letter-spacing: -1.2px; }
.cq em { font-style: normal; font-weight: 900; border-bottom: 3px solid #000; padding-bottom: 2px; }
.ctabox { background: #000; border-radius: 14px; padding: 32px 36px; }
.cta-hl { font-size: 17px; font-weight: 600; color: rgba(255,255,255,0.9); margin-bottom: 14px; line-height: 1.5; }
.cta-body { font-size: 17px; line-height: 1.75; color: rgba(255,255,255,0.75); }
.cta-link { font-family: 'Inter', sans-serif; font-size: 12px; color: rgba(255,255,255,0.5); margin-top: 18px; letter-spacing: 1px; }

/* ── Photo embed ── */
.photo-wrap { position: relative; width: 100%; border-radius: 10px; overflow: hidden; margin-bottom: 18px; flex-shrink: 0; }
.photo-wrap img { width: 100%; height: 100%; object-fit: cover; object-position: center 30%; display: block; }
.photo-caption { position: absolute; bottom: 0; left: 0; right: 0; background: linear-gradient(transparent, rgba(0,0,0,0.72)); padding: 28px 22px 16px; color: rgba(255,255,255,0.75); font-size: 12px; font-family: 'Inter', sans-serif; letter-spacing: 1.5px; text-transform: uppercase; }

/* ── 3-col stat row ── */
.stat3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px; margin-bottom: 16px; }
.s3c { background: #F5F5F5; border-radius: 10px; padding: 18px; }
.s3c .s3n { font-size: 32px; font-weight: 900; font-family: 'Inter', sans-serif; color: #000; line-height: 1; letter-spacing: -1.5px; margin-bottom: 6px; }
.s3c .s3l { font-size: 13px; color: #666; line-height: 1.5; }

/* ── Info/disclaimer box ── */
.infobox { background: #000; border-radius: 10px; padding: 20px 24px; display: flex; gap: 14px; align-items: flex-start; }
.infobox-icon { font-size: 18px; flex-shrink: 0; }
.infobox-text { font-size: 15px; line-height: 1.7; color: rgba(255,255,255,0.72); }
.infobox-text strong { color: #fff; }
</style>
</head>
<body>

<!-- 逐张生成幻灯片，结构模板如下 -->

<!-- ══ LIGHT 幻灯片（内容页）══
<div class="slide-label">Slide NN / 标题</div>
<div class="slide">
  <div class="chrome">
    <div class="dots"><div class="dot dr"></div><div class="dot dy"></div><div class="dot dg"></div></div>
    <div class="urlbar"><span class="lk">🔒</span><span>superlinear.academy{article_path}</span></div>
  </div>
  <div class="sh">
    <img src="{LOGO_PATH}" class="logo">
    <span class="bc">{section} <b>› {article_title}</b></span>
  </div>
  <div class="content">
    <div class="pad">
      ... 内容 ...
    </div>
  </div>
  <div class="sf">
    <span class="sfurl">完整文章及深度讨论见 superlinear.academy — 下载 App 加入社区</span>
    <img src="{LOGO_PATH}" class="sflogo">
  </div>
</div>
-->

<!-- ══ DARK 封面：class="slide s-cover"，chrome/sh/sf 加 .dk，logo 加 style="filter:invert(1)" ══ -->
<!-- ══ DARK 结语：class="slide s-close"，chrome/sh/sf 加 .dk，logo 加 style="filter:invert(1)" ══ -->

</body>
</html>
```

---

### Step 3 — 生成截图脚本并运行

在 `/tmp/slides-screenshot/` 创建脚本，首次使用时安装依赖：

```javascript
// /tmp/slides-screenshot/screenshot.js
const puppeteer = require('puppeteer-core');
const path = require('path');
const fs = require('fs');

const CHROME_PATHS = [
  '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
  '/Applications/Chromium.app/Contents/MacOS/Chromium',
  'C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe',
  '/usr/bin/google-chrome',
];
const CHROME = CHROME_PATHS.find(p => fs.existsSync(p));
if (!CHROME) throw new Error('未找到 Chrome，请确认已安装');

const HTML = process.argv[2];
const OUT  = process.argv[3];

(async () => {
  fs.mkdirSync(OUT, { recursive: true });
  const browser = await puppeteer.launch({
    executablePath: CHROME,
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox', '--allow-file-access-from-files', '--disable-web-security']
  });
  const page = await browser.newPage();
  await page.setViewport({ width: 1400, height: 900, deviceScaleFactor: 2 });
  await page.goto(HTML, { waitUntil: 'networkidle0', timeout: 30000 });
  await new Promise(r => setTimeout(r, 3000));
  const slides = await page.$$('.slide');
  console.log(`找到 ${slides.length} 张幻灯片`);
  for (let i = 0; i < slides.length; i++) {
    const num = String(i + 1).padStart(2, '0');
    const file = path.join(OUT, `slide-${num}.png`);
    await slides[i].screenshot({ path: file, type: 'png' });
    console.log(`  ✓ slide-${num}.png`);
  }
  await browser.close();
  console.log(`\n完成！${slides.length} 张 PNG 已保存至：\n${OUT}`);
})();
```

执行：
```bash
# 首次使用：
mkdir -p /tmp/slides-screenshot && cd /tmp/slides-screenshot && npm init -y && npm install puppeteer-core

# 每次截图：
node /tmp/slides-screenshot/screenshot.js "file://{html_absolute_path}" "{output_dir}"
```

检查 `/tmp/slides-screenshot/node_modules/puppeteer-core` 是否存在，不存在则先安装。

---

### Step 4 — 完成报告

告知用户：
- 输出目录路径
- 生成的幻灯片张数
- 每张 PNG 的实际尺寸（应为 2160×2700px @2x）

---

## 设计决策（写 HTML 时遵守）

| 规则 | 原因 |
|------|------|
| 幻灯片 1080×1350px | 小红书 4:5 最优比例 |
| deviceScaleFactor: 2 | 输出 2160×2700px 高清 |
| 正文 19px / 行高 1.85 | 手机端清晰可读的底线 |
| 强调只用粗体+下划线 | 黑底色块视觉过重，影响阅读节奏 |
| `.lbox` 左黑线+浅灰底 | 结构化信息的轻量呈现 |
| footer 固定引流文案 | 每张传播都带回流 |
| 封面深色/内容白/结语浅灰 | 视觉节奏，避免单调 |
| PingFang SC + Noto Sans SC + Inter | PingFang SC 中文更清晰易读，Noto 作备用，Inter 负责数字/英文 |
| 页尾文案不带外链 | 小红书限流外链，改用「应用商店搜索 Superlinear Academy」引导动作 |
| Closing CTA 标题固定 | 「Superlinear Academy，AI 先行者的赛博风水宝地」，下方引导搜索 App，不贴 URL |

---

## 安装

```bash
curl -o ~/.claude/skills/article-to-slides.md \
  https://raw.githubusercontent.com/sunyuzheng/xhs-article-slides/main/article-to-slides.md
```
