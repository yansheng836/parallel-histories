# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

**平行历史** —— 中外历史对照时间轴。一个**单文件、零依赖**的静态网页：`index.html` 包含全部 HTML、CSS 和 JS。没有构建步骤、没有包管理器、没有测试框架。

- 线上地址：<https://yansheng836.github.io/parallel-histories/>
- 仓库：<https://github.com/yansheng836/parallel-histories>
- 内容语言：中文优先（页面内容全部为中文），文档中英双语

**核心约束：不要引入依赖、构建工具或框架。** 零依赖是设计决策，不是技术债。不要添加 npm、打包器、CDN 引用或 CSS 框架。

## 常用命令

```bash
# 本地预览（改完刷新即可，无热更新）
python -m http.server 8000        # 然后访问 http://localhost:8000

# 直接用浏览器打开也行
start index.html                  # Windows

# 语法校验（改动后建议执行）
node --check <(sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d')   # JS
python -c "import yaml,glob; [yaml.safe_load(open(f,encoding='utf-8')) for f in glob.glob('.github/**/*.yml',recursive=True)]"  # YAML

# 结构自检：分期色带引用是否都指向存在的刻度 id
python -c "
import re,io
s=io.open('index.html',encoding='utf-8').read()
ids=set(re.findall(r'id=\"([^\"]+)\"',s))
refs=re.findall(r'data-(?:from|to)=\"#([^\"]+)\"',s)
print('broken:',[r for r in refs if r not in ids])"
```

提交前需要在**桌面端 1280px** 和**手机端 375px** 两种宽度下各看一遍。

## 架构

### 时间轴的数据结构

页面是「文档流驱动」的：内容的**书写顺序就是时间顺序**，没有数据数组、没有 JS 渲染。滚动时从上到下即为从古至今。这意味着新增事件必须插在正确的位置。

四种顶层块，按需交替出现：

| 块 | 作用 |
|----|------|
| `.era-cn` / `.era-west` | 分期色带（两侧竖排标签），用 `data-from`/`data-to` 锚定到刻度 |
| `.tick` | 时间刻度（中间的年份胶囊），带 `id` 供色带引用 |
| `.row` | 一组左右对照，内含 `.card.cn` + `.card.west` |
| `.link-note` | 「中外联动」卡片，独立居中，标记两个文明真正交汇的节点 |

一个 `.tick` 可以跟多个 `.row`（表示同一时期的多个事件）。`.row` 内某一侧可以没有对应内容。

### 分期色带的定位机制

**这是最容易出错的部分。** 色带在 HTML 里是 `position:absolute`，位置由 JS 在运行时计算：

1. `layoutBands()` 读取 `data-from` / `data-to` 指向的刻度元素，用 `getBoundingClientRect()` 算出相对 `.timeline` 的 top，赋给色带的 `style.top` / `style.height`
2. `data-to` 省略时，色带延伸到时间轴末尾（`tl.scrollHeight`）
3. 手机端（`min-width:769px` 不匹配）时清空内联定位，色带退回文档流，变成横向条（`.era-west` 在手机端 `display:none`，因为已折成单列）

触发时机有四个：`load`、`resize`、`document.fonts.ready`、以及 `ResizeObserver` 观察 `.timeline`。字体加载会影响高度，所以第四个触发条件不可省。

**因此：`data-from` / `data-to` 必须指向存在的 `.tick` id。** 指向不存在的 id 会被静默跳过（`if(!from)return;`），色带不显示且不报错。id 命名约定：`t-` + 简短标识（`t-tang`、`t-1840`）。色带声明通常写在它所锚定的起始刻度**之前**，与阅读顺序一致。

### 左右对照布局

桌面端 `.row` 是 flex，中间留出中轴。卡片宽度是 `calc(50% - 76px)`，`.row` 左右各留 48px 给色带凹槽。卡片用 `::after` 画 28px 连接线接到中轴。

手机端（≤768px）`.row` 改为 `display:block`，中轴移到左侧 14px，卡片用 `::before` 画三角箭头、`.node` 圆点绝对定位到左侧轴上。**布局靠 CSS 媒体查询切换，没有 JS 参与。**

### 入场动画

`IntersectionObserver`（threshold 0.1）在卡片进入视口时把 `opacity` 设为 1、`transform` 设为 `none`，然后 `unobserve`。初始状态由 JS 设置为 `opacity:0`——所以**禁用 JS 时卡片不会显示**。这是已知取舍。

### CSS 组织

单块 `<style>`，顺序为：`:root` 变量 → 全局重置 → `header` → `.gh-corner` → `.legend` → `.timeline`/`.axis` → `.era-*` → `.tick` → `.row`/`.card` → `.link-note` → `footer` → `@media (max-width:768px)`。

颜色一律走 `:root` 变量：`--cn`（中国红，左侧卡片）、`--west`（蓝，右侧卡片）、`--link`（紫，联动卡片）。**不要硬编码颜色值**，新增颜色先加变量。

## 内容规范

页面内容是历史材料，改动时有额外约束（详见 `CONTRIBUTING.md`）：

- **年份格式**：公元前用 `前221年`；区间用中文破折号 `（618—907）`；精确日期 `1911.10.10`；模糊年代 `约前2000—前1200`
- **卡片三段结构**：`<h3>标题 <span class="y">（年份）</span></h3>` → `<div class="fig">人物细节</div>` → 无标签的意义概括句
- **`.link-note` 只用于真实的相互影响**，不能用来表示"同一时期"。丝绸之路、怛罗斯之战、鸦片战争、十月革命→中共成立这类才算联动
- **史实宁可标注存疑，不要凭印象写年份**；生卒不确定时写在位年份
- 近现代史表述以中国现行中学历史教材通行口径为基础，用词保持中性

## 提交与发布

- 提交信息格式：`type: 中文描述`，type 取 `feat` / `fix` / `docs` / `refactor` / `style` / `test` / `ci` / `chore`
- 推送 `main` 会由 `.github/workflows/deploy-pages.yml` 自动部署到 GitHub Pages（约 1 分钟）。改动生效后可以这样验证：

```bash
curl -sS -o /dev/null -w "%{http_code}\n" https://yansheng836.github.io/parallel-histories/
```

- 改动史实内容时同步更新 `CHANGELOG.md` 的 `[Unreleased]` 段

## 已知待办

- `docs/preview.png` 预览图尚未提供。`README.md` / `README.en.md` 中保留了 TODO 注释说明截图步骤（1280px 宽，截「隋唐盛世」到「明清」段），补图后需删掉注释和提示行
- 不要提交 `.playwright-mcp/`、`_svgcheck.html` 等本地调试产物（已在 `.gitignore` 中）
