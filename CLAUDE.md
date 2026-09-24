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

# 结构自检：比例尺坐标（data-date / data-start / data-end）是否合法
python -c "
import re,io
s=io.open('index.html',encoding='utf-8').read()
ticks=re.findall(r'class=\"tick\" id=\"(t-[^\"]+)\"[^>]*',s)
missing=[t for t in ticks if 'data-date=' not in dict(re.findall(r'id=\"(t-[^\"]+)\"(.*?)>',s)).get(t,'')]
print('ticks:',len(ticks),'missing data-date:',missing)
bands=re.findall(r'class=\"era-(?:cn|west)\" data-start=\"(-?[\d.]+)\"(?: data-end=\"(-?[\d.]+)\")?',s)
bad=[(a,c) for a,c in bands if c and float(c)<float(a)]
print('bands:',len(bands),'invalid start>end:',bad)"
```

提交前需要在**桌面端 1280px** 和**手机端 375px** 两种宽度下各看一遍。

## 架构

### 时间轴的数据结构

页面是「文档流驱动」的：内容的**书写顺序就是时间顺序**，没有数据数组、没有 JS 渲染。滚动时从上到下即为从古至今。**新增事件必须插在时间正确的位置**——书写顺序在手机端（单列文档流）直接决定视觉顺序；在桌面端它决定内容归属到哪个刻度区间（进而影响该区间的膨胀高度），所以也仍要插对。

四种顶层块，按需交替出现：

| 块 | 作用 | 关键属性 |
|----|------|----------|
| `.era-cn` / `.era-west` | 分期色带（两侧竖排标签），`data-from`/`data-to` 锚定刻度（供手机端文档流），`data-start`/`data-end` 提供真实年份（供桌面端比例尺） | `data-start` `data-end`（可省） |
| `.tick` | 时间刻度（中间的年份胶囊），带 `id` 供色带引用；`data-date` 提供该刻度的真实年份 | `data-date` |
| `.row` | 一组左右对照，内含 `.card.cn` + `.card.west` | — |
| `.link-note` | 「中外联动」卡片，独立居中，标记两个文明真正交汇的节点 | — |

一个 `.tick` 可以跟多个 `.row`（表示同一时期的多个事件）。`.row` 内某一侧可以没有对应内容。

**桌面端（≥769px）的时间排布不再由文档流决定，而是由 `data-date` 等比铺开**——见下节「时间比例尺」。`data-date` / `data-start` / `data-end` 是这个机制的权威数据源；`data-from` / `data-to` 只在手机端文档流里起作用。

### 时间比例尺（桌面端节点长度的核心逻辑）

**这是新增/删除节点时最容易踩坑的部分。** 桌面端每个时间段的纵向长度按真实时间跨度等比铺开：`前2070—前1046`（1024 年）会明显比 `1840—1911`（71 年）长得多，「夏商·王朝之始」色带因此会是一大片真实空白——这正是设计意图，**空白本身即信息**。

数据源（全部是十进制年份，公元前为**负数**；模糊刻度取约定值，见设计文档 3.1/3.2）：

| 属性 | 加在哪 | 含义 | 缺了会怎样 |
|------|--------|------|------------|
| `data-date` | 每个 `.tick` | 该刻度的真实年份（`-2070`、`1911`、`2012`） | 桌面端布局**中止**，Console `warn`，整段退回文档流（不静默失败） |
| `data-start` | 每条 `.era-*` | 色带起年 | 该色带跳过定位，`warn` |
| `data-end` | 每条 `.era-*`（可省） | 色带止年；省略时延伸到轴尾 | 省略是合法的（延伸到底） |

**新增节点 / 色带时必须同步写上对应属性**，否则桌面端比例尺会因 `data-date` 缺失或时间未递增而中止（见 6.2 降级）。删除节点时把对应的 `.tick`（连同它的 `data-date`）和引用它的色带属性一起删干净。

算法（`layoutTimeline()`，运行时双 pass 计算，写内联 `absolute` 定位；CSS **不预设** `position:absolute`，所以禁 JS 时自动回退文档流）：

1. **归组**：按 DOM 顺序把每个 `.row` / `.link-note` 归属到它**前面最近**的 `.tick`。校验 `data-date` 必须随 DOM 顺序严格递增，违反即 `warn` + 中止。
2. **Pass 1 · 反解比例密度 `k`**：每个区间的内容需求 `need = 胶囊高 + 区间内所有卡片高度和 + 间距`；解方程 `Σ max(span·k, need) + 轴尾 = H_TARGET(15000px)` 用二分法求出 `k`（px/年）。整页高度由此目标决定，密集区间自动「膨胀」。
3. **Pass 2 · 写定位**：区间实际高度取 `max(span·k, need)`，逐段累加得到每个刻度的 `top`；`need > span·k` 的区间判为**膨胀**，在其**中点**插入 `.axis-break`（`⧗ 此段比例已放大`）标注。
4. **色带**：用 `date→y` 分段线性映射（非膨胀段斜率 `k`，膨胀段斜率 `need/span`）直接由 `data-start`/`data-end` 算出 `top`/`height`，**不再读刻度 DOM**。

调参常量（都在 `layoutTimeline()` 顶部，`index.html`）：`H_TARGET`（目标整页高度）、`K_MIN`（`k` 下限，防先秦段被压没）、`TICK_GAP`（胶囊/内容间距）。

### 分期色带的手机端行为

**（手机端 ≤768px 没有比例尺，色带退回文档流）** 手机端时 `layoutTimeline()` 清除所有内联定位，色带变成横向条，`.era-west` 因单列折排 `display:none`。此时色带的**书写顺序**（跟随 `data-from` 锚定的刻度位置）决定它出现在哪一段，因此新增色带仍要写在它锚定的起始刻度**之前**，与阅读顺序一致。

### 分期色带的定位机制（桌面端）

色带在 HTML 里是 `position:absolute`，位置由 `layoutTimeline()` 按上节的比例尺算法运行时计算，触发时机四个：`load`、`resize`、`document.fonts.ready`、以及 `ResizeObserver` 观察 `.timeline`。字体加载会影响卡片高度（进而影响 `need` 与 `k`），所以第四个触发条件不可省。

**因此：`data-from` / `data-to` 必须指向存在的 `.tick` id**（手机端文档流用），`data-start` / `data-end` 必须是合法十进制年份且 `start ≤ end`（桌面端比例尺用）。指向不存在的 id 或非法年份会被跳过并 `warn`。id 命名约定：`t-` + 简短标识（`t-tang`、`t-1840`）。

### 左右对照布局

桌面端 `.row` 是 flex，中间留出中轴。卡片宽度是 `calc(50% - 76px)`，`.row` 左右各留 48px 给色带凹槽。卡片用 `::after` 画 28px 连接线接到中轴。`.link-note` 在桌面端被比例尺定位后用 `left:50% + translateX(-50%)` 水平居中（否则它会贴左缘，见 `placeItem()`）。

手机端（≤768px）`.row` 改为 `display:block`，中轴移到左侧 14px，卡片用 `::before` 画三角箭头、`.node` 圆点绝对定位到左侧轴上。手机端布局**不经过比例尺**，靠 CSS 媒体查询 + 文档流切换，`layoutTimeline()` 直接清除内联定位后 return。

### 入场动画

`IntersectionObserver`（threshold 0.1）在卡片进入视口时把 `opacity` 设为 1、`transform` 设为 `none`，然后 `unobserve`。初始状态由 JS 设置为 `opacity:0`——所以**禁用 JS 时卡片不会显示**。这是已知取舍。注意：比例尺定位只改 `top`/`position`，不改卡片的 `opacity`/`transform`，两者互不耦合，禁 JS 时页面退回文档流、内容照常可读，只是没有比例尺。

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

- **改动史实内容时不需要更新 `CHANGELOG.md`。** `[Unreleased]` 段只在正式发布时才整理（本项目尚未发布过版本），日常勘误不往里记

## 已知待办

- `docs/preview.png` 预览图尚未提供。`README.md` / `README.en.md` 中保留了 TODO 注释说明截图步骤（1280px 宽，截「隋唐盛世」到「明清」段），补图后需删掉注释和提示行
- 不要提交 `.playwright-mcp/`、`_svgcheck.html` 等本地调试产物（已在 `.gitignore` 中）
