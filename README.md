# article-to-slides

**Claude Code skill（Superlinear Academy 专属版）** — 将文章转换为小红书「网页截图风」幻灯片，自动输出高清 PNG。

品牌信息（logo、网站 URL、引流文案）已内置，调用时只需提供文章内容，无需重复配置。

---

## 效果预览

每张幻灯片模拟网页截图外观：
- macOS 浏览器地址栏（含真实 URL）
- 品牌 logo + 面包屑导航
- 正文内容（封面深色 / 内容白色 / 结语浅灰）
- 底部引流 footer
- 输出尺寸：**2160×2700px**（@2x Retina，相当于 1080×1350 逻辑像素，小红书 4:5 最优比例）

典型幻灯片结构：封面 → 背景/关于 → TLDR → 数据悖论 → 瓶颈/问题 → 案例 → 原则/框架 → 行动建议 → 结语引流

---

## 安装

```bash
curl -o ~/.claude/skills/article-to-slides.md \
  https://raw.githubusercontent.com/sunyuzheng/xhs-article-slides/main/article-to-slides.md
```

安装后在 Claude Code 任意项目中输入：

```
/article-to-slides
```

---

## 使用流程

1. 运行 `/article-to-slides`
2. Claude 只会询问：
   - 文章内容（URL 或直接粘贴全文）
   - 文章路径（如 `/c/ai-resources/my-article`，可从 URL 自动解析）
   - 输出目录（可选，默认 `~/Desktop/slides-png/`）
   - **品牌 logo、网站 URL、引流文案均已内置，不会重复询问**
3. Claude 自动：
   - 解析文章结构，规划 10–16 张幻灯片
   - 生成 `slides.html`（完整设计系统）
   - 安装 `puppeteer-core`（首次使用时）
   - 批量截图输出 `slide-01.png` … `slide-NN.png`

---

## 依赖

- **Node.js** + **npm**（用于运行截图脚本）
- **Google Chrome** 或 **Chromium**（已安装在本机）
- 无需 Puppeteer 付费订阅，使用 `puppeteer-core` + 本机 Chrome

---

## 设计规范

| 项目 | 规格 |
|------|------|
| 幻灯片尺寸 | 1080×1350px（输出 @2x = 2160×2700px） |
| 字体 | Inter（英文/数字）+ Noto Sans SC（中文） |
| 正文字号 | 19px / 行高 1.85 |
| 强调方式 | `font-weight: 700-900`，下划线，避免黑底色块 |
| 配色 | 黑白灰（无彩色），深色封面 + 白色内容 + 浅灰结语 |
| 浏览器 chrome | macOS 风格，红黄绿三点 + 地址栏 |

---

## 本地开发 / 自定义

skill 文件本身就是 Markdown，直接编辑 `article-to-slides.md` 即可定制：
- 修改配色方案
- 调整字号和间距
- 增加新的幻灯片类型
- 换用自己的设计系统

---

## License

MIT
