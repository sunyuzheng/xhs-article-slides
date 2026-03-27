---
name: article-to-slides
description: 将长文章转换为小红书「网页截图风」幻灯片，自动输出 1080×1350px PNG 文件，适合文章分发和引流。
---

# Article → 小红书「网页截图风」幻灯片生成器

将一篇文章转换为一套专业的小红书竖版幻灯片组。每张幻灯片模拟网页截图外观（带浏览器地址栏、品牌 logo、引流 footer），用 puppeteer-core 批量输出高清 PNG。

## 使用方式

```
/article-to-slides
```

启动后 Claude 会询问必要信息，然后全自动完成生成和截图。

---

## Claude 执行指令

当用户调用此 skill 时，按以下步骤执行：

### Step 0 — 收集输入

向用户询问（未提供的才问）：

1. **文章内容**：URL 或直接粘贴的文章全文
2. **品牌 logo 路径**：本地图片绝对路径（可选，无则略去 logo）
3. **品牌网站 URL**：例如 `superlinear.academy`（用于 address bar 和 footer 引流）
4. **文章在网站上的路径**：例如 `/c/ai-resources/ai-native-orgs`（可选）
5. **输出目录**：PNG 保存位置（默认与 HTML 文件同目录下的 `slides-png/`）
6. **引流文案**：footer 底部文字（可选，默认 `完整文章见 {brand_url} — 下载 App 加入社区`）

如果用户提供了 URL，使用 WebFetch 获取文章正文。

### Step 1 — 规划幻灯片结构

将文章内容切分为以下标准幻灯片类型（根据文章实际内容取舍，一般 10–16 张）：

| 类型 | 说明 |
|------|------|
| **封面 (Cover)** | 深色背景，大字标题 + 副标题 + 核心论点引用框 + 来源声明 |
| **关于/背景** | 可选。作者/机构背景，为什么有资格说这个，可嵌入照片 |
| **TLDR** | 4–6 条核心观点，左黑边框卡片列表 |
| **数据/悖论** | 统计数字卡片网格 + 结论框 |
| **问题/瓶颈** | 每个瓶颈一张（左边栏 label + 标题 + 正文 + 数据卡 + quote box） |
| **案例** | 每个案例一张（来源标注 + 案例标题 + 逻辑框 + 角色列表） |
| **原则/框架** | 每个原则一张（大背景数字 + 标题 + 正文 + 框架组件） |
| **行动建议** | 编号列表，每条含标题 + 说明 |
| **结语 (Closing)** | 浅灰背景，大字金句（下划线强调，不用黑底方块）+ 引流 CTA 黑框 |

**内容处理原则**：
- 正文字体是主角，不要过度使用「强调块」
- 每张幻灯片字数控制在可清晰阅读范围内（不强求填满）
- 强调方式优先用：`font-weight: 700-900`，次选下划线，避免背景色块

### Step 2 — 生成 HTML 文件

在输出目录的父目录写入 `slides.html`，使用以下完整设计系统：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>{文章标题} — Slides</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&family=Noto+Sans+SC:wght@300;400;500;700;900&display=swap" rel="stylesheet">
<style>
/* ── Reset ── */
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  background: #C4C4C4;
  padding: 48px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 48px;
  font-family: 'Noto Sans SC', 'Inter', -apple-system, sans-serif;
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

/* ── Slide container: 1080×1350px (小红书 4:5) ── */
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

/* ── Browser Chrome (macOS style) ── */
.chrome { background: #2C2C2E; padding: 13px 20px 11px; display: flex; flex-direction: column; gap: 9px; flex-shrink: 0; }
.chrome.dk { background: #1A1A1C; }
.dots { display: flex; gap: 7px; align-items: center; }
.dot { width: 12px; height: 12px; border-radius: 50%; }
.dr { background: #FF5F56; } .dy { background: #FFBD2E; } .dg { background: #27C93F; }
.urlbar { background: #3C3C3E; border-radius: 7px; padding: 6px 14px; display: flex; align-items: center; gap: 8px; color: #E0E0E5; font-size: 12px; font-family: 'Inter', sans-serif; }
.urlbar .lk { font-size: 11px; opacity: 0.55; }

/* ── Site Header ── */
.sh { background: #fff; border-bottom: 1px solid #EAEAEA; padding: 15px 36px; display: flex; align-items: center; justify-content: space-between; flex-shrink: 0; height: 72px; }
.sh.dk { background: #0F0F0F; border-bottom: 1px solid #1E1E1E; }
.logo { height: 52px; object-fit: contain; }
.bc { font-size: 11px; color: #BBBBBB; font-family: 'Inter', sans-serif; }
.bc b { color: #555; font-weight: 600; }
.sh.dk .bc b { color: #777; }

/* ── Site Footer ── */
.sf { background: #F4F4F4; border-top: 1px solid #E8E8E8; padding: 13px 36px; display: flex; align-items: center; justify-content: space-between; flex-shrink: 0; height: 68px; }
.sf.dk { background: #090909; border-top: 1px solid #1A1A1A; }
.sfurl { font-size: 12px; color: #999; font-family: 'Inter', sans-serif; }
.sf.dk .sfurl { color: #555; }
.sflogo { height: 30px; object-fit: contain; opacity: 0.65; }
.sf.dk .sflogo { filter: invert(1); opacity: 0.3; }

.content { flex: 1; overflow: hidden; }

/* ── Shared content pad ── */
.pad { padding: 44px 60px 36px; height: 100%; display: flex; flex-direction: column; background: #fff; }
.ptag { font-size: 10px; font-family: 'Inter', sans-serif; font-weight: 700; letter-spacing: 3px; text-transform: uppercase; color: #AAAAAA; margin-bottom: 14px; display: flex; align-items: center; gap: 10px; }
.ptag::before { content: ''; display: block; width: 18px; height: 1.5px; background: #CCC; }
.ph1 { font-size: 42px; font-weight: 900; line-height: 1.18; color: #050505; letter-spacing: -1px; margin-bottom: 28px; }
.ph1 sub { font-size: 20px; color: #AAA; font-weight: 400; display: block; letter-spacing: 0; vertical-align: baseline; margin-bottom: 6px; }

/* ── Body text (主角) ── */
.pbody { font-size: 19px; line-height: 1.85; color: #333; }
.pbody p + p { margin-top: 18px; }
.pbody strong { font-weight: 700; color: #0A0A0A; }
.dv { width: 100%; height: 1px; background: #EAEAEA; margin: 20px 0; }

/* ── Logic box: 左边黑线 + 浅灰底（不用黑底，避免视觉过重）── */
.lbox { background: #F6F6F6; border-left: 4px solid #000; border-radius: 0 10px 10px 0; padding: 22px 28px; margin-bottom: 16px; }
.lbox .lbl { font-size: 10px; font-family: 'Inter', sans-serif; letter-spacing: 2px; text-transform: uppercase; color: #AAAAAA; margin-bottom: 8px; }
.lbox .txt { font-size: 18px; line-height: 1.75; color: #1A1A1A; }
.lbox .txt strong { color: #000; font-weight: 800; }

/* ── Quote box ── */
.qbox { border-left: 4px solid #000; padding: 18px 24px; background: #F7F7F7; border-radius: 0 8px 8px 0; margin-top: 18px; }
.qt { font-size: 19px; line-height: 1.72; color: #1A1A1A; font-style: italic; }
.qs { font-size: 12px; color: #AAA; margin-top: 8px; font-style: normal; font-family: 'Inter', sans-serif; }

/* ── TLDR rows ── */
.tlist { display: flex; flex-direction: column; gap: 12px; margin-top: 4px; }
.trow { display: flex; align-items: stretch; background: #F6F6F6; border-radius: 8px; overflow: hidden; border-left: 3.5px solid #000; }
.trow-inner { padding: 17px 22px; font-size: 18px; line-height: 1.65; color: #1A1A1A; }
.trow-inner strong { font-weight: 700; color: #000; }

/* ── Stat grid ── */
.sgrid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 16px; }
.sc { border-radius: 12px; padding: 28px 26px; background: #F4F4F4; }
.sc.blk { background: #000; }
.sc .sn { font-size: 58px; font-weight: 900; font-family: 'Inter', sans-serif; letter-spacing: -3px; line-height: 1; color: #000; margin-bottom: 8px; }
.sc.blk .sn { color: #fff; }
.sc .sl { font-size: 14px; color: #777; line-height: 1.55; }
.sc.blk .sl { color: rgba(255,255,255,0.5); }
.sc .ss { font-size: 11px; color: #CCC; font-family: 'Inter', sans-serif; margin-top: 5px; }
.sc.blk .ss { color: rgba(255,255,255,0.22); }

/* ── Data/paradox box ── */
.pdx { background: #F0F0F0; border-radius: 12px; padding: 26px 34px; text-align: center; }
.pdx-q { font-size: 26px; font-weight: 800; color: #000; line-height: 1.38; }
.pdx-note { font-size: 13px; color: #999; margin-top: 10px; font-style: italic; line-height: 1.5; }

/* ── Data card (bottleneck/research) ── */
.bcard { border-radius: 12px; padding: 22px 28px; background: #F5F5F5; margin-bottom: 14px; }
.bn { font-size: 10px; font-family: 'Inter', sans-serif; letter-spacing: 2.5px; text-transform: uppercase; font-weight: 700; color: #BBB; margin-bottom: 8px; }
.bt { font-size: 20px; font-weight: 800; color: #000; margin-bottom: 10px; }
.bb { font-size: 17px; line-height: 1.78; color: #444; }
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
.ri-desc { font-size: 15.5px; line-height: 1.65; color: #555; }

/* ── Principle slide ── */
.p-bg-num { font-size: 96px; font-weight: 900; font-family: 'Inter', sans-serif; color: #EBEBEB; line-height: 1; letter-spacing: -5px; margin-bottom: -18px; user-select: none; }
.trait-rows { display: flex; flex-direction: column; gap: 10px; margin-top: 2px; }
.tr { display: grid; grid-template-columns: 170px 1fr; gap: 14px; background: #F5F5F5; border-radius: 9px; padding: 15px 18px; }
.trn { font-size: 13px; font-family: 'Inter', sans-serif; font-weight: 800; color: #000; padding-top: 1px; }
.trd { font-size: 15.5px; line-height: 1.62; color: #555; }

/* ── Framework layers (3-step) ── */
.ctx-rows { display: flex; flex-direction: column; gap: 13px; margin-top: 4px; }
.cr { border-radius: 10px; padding: 22px 26px; background: #F5F5F5; display: flex; gap: 18px; }
.cr-n { font-size: 30px; font-weight: 900; font-family: 'Inter', sans-serif; color: #D8D8D8; line-height: 1; width: 34px; flex-shrink: 0; }
.cr-t { font-size: 17px; font-weight: 700; color: #000; margin-bottom: 5px; }
.cr-b { font-size: 15.5px; line-height: 1.68; color: #555; }

/* ── Action steps ── */
.arows { display: flex; flex-direction: column; gap: 15px; margin-top: 4px; }
.ar { display: flex; gap: 18px; align-items: flex-start; }
.anum { width: 38px; height: 38px; background: #000; color: #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 16px; font-weight: 800; font-family: 'Inter', sans-serif; flex-shrink: 0; margin-top: 2px; }
.at { font-size: 20px; font-weight: 700; color: #000; margin-bottom: 5px; }
.ab { font-size: 17px; line-height: 1.72; color: #444; }

/* ── Cover slide ── */
.s-cover .content { background: #080808; padding: 56px 72px 48px; display: flex; flex-direction: column; }
.cover-ey { font-size: 11px; font-family: 'Inter', sans-serif; letter-spacing: 3.5px; text-transform: uppercase; color: #3A3A3A; margin-bottom: 0; }
.cover-main { flex: 1; display: flex; flex-direction: column; justify-content: center; }
.cover-title { font-size: 66px; font-weight: 900; line-height: 1.08; color: #FFFFFF; letter-spacing: -2px; margin-bottom: 20px; }
.cover-sub { font-size: 22px; color: #4A4A4A; font-weight: 400; margin-bottom: 44px; line-height: 1.45; }
.cover-box { border-left: 3px solid #FFFFFF; padding: 24px 30px; background: rgba(255,255,255,0.04); border-radius: 0 10px 10px 0; margin-bottom: 32px; }
.cover-bt { font-size: 20px; line-height: 1.75; color: #999; font-weight: 400; }
.cover-bt strong { color: #fff; font-weight: 700; }
.cover-source { display: flex; align-items: center; gap: 14px; padding: 14px 18px; background: rgba(255,255,255,0.03); border: 1px solid #1E1E1E; border-radius: 8px; }
.cover-source-dot { width: 6px; height: 6px; background: #27C93F; border-radius: 50%; flex-shrink: 0; }
.cover-source-text { font-size: 13px; color: #555; font-family: 'Inter', sans-serif; line-height: 1.5; }
.cover-source-text b { color: #888; font-weight: 600; }
.cover-meta { font-size: 11px; font-family: 'Inter', sans-serif; color: #2A2A2A; letter-spacing: 2px; text-transform: uppercase; margin-top: 28px; }

/* ── Closing slide ── */
.s-close .content { background: #F8F8F8; padding: 64px 72px; display: flex; flex-direction: column; justify-content: center; }
.cq { font-size: 44px; font-weight: 900; line-height: 1.26; color: #000; margin-bottom: 44px; letter-spacing: -1.2px; }
.cq em { font-style: normal; font-weight: 900; border-bottom: 3px solid #000; padding-bottom: 2px; }
.ctabox { background: #000; border-radius: 14px; padding: 32px 36px; }
.cta-hl { font-size: 17px; font-weight: 600; color: rgba(255,255,255,0.9); margin-bottom: 14px; line-height: 1.5; }
.cta-body { font-size: 15px; line-height: 1.75; color: rgba(255,255,255,0.52); }
.cta-link { font-family: 'Inter', sans-serif; font-size: 12px; color: rgba(255,255,255,0.28); margin-top: 18px; letter-spacing: 1px; }

/* ── Photo embed (for background/about slide) ── */
.photo-wrap { position: relative; width: 100%; border-radius: 10px; overflow: hidden; margin-bottom: 18px; flex-shrink: 0; }
.photo-wrap img { width: 100%; height: 100%; object-fit: cover; object-position: center 30%; display: block; }
.photo-caption { position: absolute; bottom: 0; left: 0; right: 0; background: linear-gradient(transparent, rgba(0,0,0,0.72)); padding: 28px 22px 16px; color: rgba(255,255,255,0.75); font-size: 12px; font-family: 'Inter', sans-serif; letter-spacing: 1.5px; text-transform: uppercase; }

/* ── 3-col stat row (for about slide) ── */
.stat3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px; margin-bottom: 16px; }
.s3c { background: #F5F5F5; border-radius: 10px; padding: 18px; }
.s3c .s3n { font-size: 32px; font-weight: 900; font-family: 'Inter', sans-serif; color: #000; line-height: 1; letter-spacing: -1.5px; margin-bottom: 6px; }
.s3c .s3l { font-size: 13px; color: #666; line-height: 1.5; }

/* ── Bottom info box (for about slide) ── */
.infobox { background: #000; border-radius: 10px; padding: 20px 24px; display: flex; gap: 14px; align-items: flex-start; }
.infobox-icon { font-size: 18px; flex-shrink: 0; }
.infobox-text { font-size: 15px; line-height: 1.7; color: rgba(255,255,255,0.72); }
.infobox-text strong { color: #fff; }
</style>
</head>
<body>

<!-- 按 Step 1 规划的结构逐张生成幻灯片 -->
<!-- 每张幻灯片结构如下（以 LIGHT 幻灯片为例）： -->

<!--
<div class="slide-label">Slide NN / 标题</div>
<div class="slide">
  <div class="chrome">
    <div class="dots"><div class="dot dr"></div><div class="dot dy"></div><div class="dot dg"></div></div>
    <div class="urlbar"><span class="lk">🔒</span><span>{brand_url}{article_path}</span></div>
  </div>
  <div class="sh">
    <img src="{logo_path}" class="logo">
    <span class="bc">{breadcrumb} <b>› {article_section}</b></span>
  </div>
  <div class="content">
    <div class="pad">
      ... 内容 ...
    </div>
  </div>
  <div class="sf">
    <span class="sfurl">{引流文案}</span>
    <img src="{logo_path}" class="sflogo">
  </div>
</div>
-->

<!-- DARK 封面：class="slide s-cover"，header/footer 加 .dk -->
<!-- LIGHT 内容：class="slide"，header/footer 不加 .dk -->
<!-- LIGHT 结语：class="slide s-close"，header/footer 加 .dk -->

</body>
</html>
```

### Step 3 — 生成截图脚本

在 `/tmp/slides-screenshot/` 创建 `screenshot.js`，并安装 `puppeteer-core`（只需执行一次）：

```javascript
// /tmp/slides-screenshot/screenshot.js
const puppeteer = require('puppeteer-core');
const path = require('path');
const fs = require('fs');

// macOS Chrome 路径（Windows/Linux 用户需修改）
const CHROME_PATHS = [
  '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
  '/Applications/Chromium.app/Contents/MacOS/Chromium',
  'C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe',
  '/usr/bin/google-chrome',
];

const CHROME = CHROME_PATHS.find(p => fs.existsSync(p));
if (!CHROME) throw new Error('未找到 Chrome，请手动设置 CHROME 路径');

const HTML  = process.argv[2];  // HTML 文件路径（file:// URL）
const OUT   = process.argv[3];  // 输出目录

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
  await new Promise(r => setTimeout(r, 3000));  // 等待字体加载
  const slides = await page.$$('.slide');
  console.log(`找到 ${slides.length} 张幻灯片`);
  for (let i = 0; i < slides.length; i++) {
    const num  = String(i + 1).padStart(2, '0');
    const file = path.join(OUT, `slide-${num}.png`);
    await slides[i].screenshot({ path: file, type: 'png' });
    console.log(`  ✓ slide-${num}.png`);
  }
  await browser.close();
  console.log(`\n完成！PNG 已保存至：\n${OUT}`);
})();
```

执行方式：
```bash
cd /tmp/slides-screenshot && npm init -y && npm install puppeteer-core
node screenshot.js "file:///path/to/slides.html" "/path/to/output-dir"
```

### Step 4 — 运行并报告

1. 写入 HTML 文件（路径与用户提供的输出目录同级）
2. 检查 `/tmp/slides-screenshot/node_modules/puppeteer-core` 是否存在，不存在则安装
3. 执行截图脚本
4. 告知用户输出目录路径和生成的幻灯片数量

---

## 设计决策说明（编写 HTML 时遵守）

| 规则 | 原因 |
|------|------|
| 幻灯片尺寸 1080×1350px | 小红书 4:5 比例最优 |
| deviceScaleFactor: 2 | 输出 2160×2700px 高清（Retina） |
| 正文 19px / 行高 1.85 | 小红书手机端清晰可读的最小字号 |
| 强调用 `font-weight: 700` + 偶尔下划线 | 黑底方块视觉过重，影响阅读节奏 |
| `.lbox` 左黑线 + 浅灰底 | 结构化信息的轻量呈现 |
| footer 全部用引流文案 | 每张传播都带回流 |
| 封面深色 / 内容白色 / 结语浅灰 | 视觉节奏感，避免全白单调 |
| Google Fonts 中英混排 | Inter（英文/数字）+ Noto Sans SC（中文）效果最佳 |

---

## 安装方式

```bash
curl -o ~/.claude/skills/article-to-slides.md \
  https://raw.githubusercontent.com/sunyuzheng/xhs-article-slides/main/article-to-slides.md
```

然后在 Claude Code 中输入 `/article-to-slides` 即可调用。
