<div align="center">

# 中外历史对照时间轴

**Parallel Histories — China & The World**

把中国史和世界史放在同一条时间轴上对照阅读。

[![在线访问](https://img.shields.io/badge/在线访问-yansheng836.github.io-2c3e50?style=flat-square)](https://yansheng836.github.io/parallel-histories/)
[![License: MIT](https://img.shields.io/badge/License-MIT-c0392b?style=flat-square)](LICENSE)
[![单文件](https://img.shields.io/badge/单文件-HTML-2471a3?style=flat-square)](#)
[![无依赖](https://img.shields.io/badge/依赖-零-27ae60?style=flat-square)](#)

[English](README.en.md) | **简体中文**

<img src="docs/preview.png" alt="中外历史对照时间轴预览" width="90%">

</div>

---

## 这是什么

一个**单文件、零依赖**的静态网页。中间是一条纵向时间轴，**左侧是中国大事，右侧是西方／世界大事**，同一时期的事件横向并排，便于对照。

时间跨度从**约公元前 2070 年（夏朝建立）**一直到**当下**，覆盖：

| 分期 | 对照内容 |
|------|----------|
| 上古三代 | 夏商周 ←→ 爱琴文明、古希腊城邦、亚历山大东征 |
| 秦汉 | 大一统帝国 ←→ 罗马共和国与罗马帝国 |
| 三国两晋南北朝 | 民族大交融 ←→ 西罗马灭亡、欧洲中世纪开始 |
| 隋唐 | 隋唐盛世 ←→ 拜占庭、阿拉伯帝国崛起 |
| 宋元 | 经济文化高峰 ←→ 十字军东征、大宪章、马可·波罗东游 |
| 明清 | 专制与闭关 ←→ 文艺复兴、大航海、宗教改革、工业革命 |
| 晚清 | 鸦片战争到辛丑条约 ←→ 资本主义制度扩展、第二次工业革命 |
| 民国 | 辛亥革命到抗战胜利 ←→ 两次世界大战、十月革命 |
| 当代 | 新中国到新时代 ←→ 冷战、全球化、多极化 |

其中还单列了**「中外联动」**卡片（紫色虚线框），标注两个文明真正发生交汇的节点，例如：

- 张骞通西域开辟**丝绸之路**，中国丝绸直抵罗马
- 工业革命 → 鸦片走私 → 虎门销烟 → **鸦片战争**
- 巴黎和会外交失败 → **五四运动**；十月革命 → 中国共产党成立
- 1971 恢复联合国席位 → 1972 尼克松访华 → 1979 中美建交

## 特性

- **零依赖**：不需要 npm、不需要构建、不需要 CDN，一个 `index.html` 就是全部
- **离线可用**：下载后双击即可打开，断网也能看
- **响应式**：桌面端左右对照布局；移动端（≤768px）自动切换为单列时间轴
- **入场动画**：使用原生 `IntersectionObserver`，滚动到视口时卡片淡入
- **分期色带**：年代分期色带自动跟随时间轴刻度定位，用 `ResizeObserver` 保持对齐
- **可打印**：打印时即为一份中外对照年表

## 快速开始

### 在线访问

<https://yansheng836.github.io/parallel-histories/>

### 本地运行

无需任何环境，三种方式任选：

```bash
# 方式一：直接打开
start index.html        # Windows
open index.html         # macOS

# 方式二：起一个本地服务（推荐，便于手机同局域网访问）
python -m http.server 8000
# 然后浏览器访问 http://localhost:8000

# 方式三：VS Code 装 Live Server 插件，右键 index.html → Open with Live Server
```

## 如何贡献

非常欢迎补充内容。这个项目**只需要你编辑 HTML**，没有构建步骤。

### 补充一条历史事件

1. 打开 `index.html`
2. 找到对应的 `.row` 块（`Ctrl+F` 搜索年份，例如 `1840年`）
3. 在 `.card.cn` 里加中国的事，在 `.card.west` 里加同期西方的事

```html
<div class="row">
  <div class="card cn"><i class="node"></i>
    <h3>事件标题 <span class="y">（起止年份）</span></h3>
    <div class="fig">关键人物、关键细节</div>
    一句话概括这件事的意义。</div>
  <div class="card west"><i class="node"></i>
    <h3>同期西方事件 <span class="y">（年份）</span></h3>
    说明。</div>
</div>
```

### 新增一个时间刻度

```html
<div class="tick" id="t-年份"><span class="year">公元年份</span><span class="era-tag l">事件名</span></div>
```

### 新增一条「中外联动」

```html
<div class="link-note"><div class="inner">
  ⚡ <b>联动主题</b>：因 → 果 → 影响。
</div></div>
```

### 样式约定

| 类名 | 用途 |
|------|------|
| `.card.cn` | 左侧中国卡片（红色系） |
| `.card.west` | 右侧西方卡片（蓝色系） |
| `.card .y` | 卡片标题里的年份，淡化显示 |
| `.card .fig` | 关键人物／细节行 |
| `.link-note` | 中外联动卡片（紫色虚线框） |
| `.era-cn` / `.era-west` | 两侧分期色带，用 `data-from` / `data-to` 指定起止刻度 |

新增分期色带时 `data-from` / `data-to` 必须指向存在的刻度 `id`，脚本会自动计算位置和高度。

### 提交 Pull Request

```bash
git clone git@github.com:yansheng836/parallel-histories.git
cd parallel-histories
git checkout -b feat/add-ming-dynasty-detail
# 编辑 index.html
git commit -m "feat: 补充明朝郑和下西洋细节"
git push -u origin feat/add-ming-dynasty-detail
```

提交信息请遵循 `type: 中文描述` 格式，type 取 `feat` / `fix` / `docs` / `refactor` / `style` / `test` / `ci` / `chore`。

详细规范见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 内容说明与免责

- 本项目是**历史学习辅助材料**，不是学术著作。事件年份与人物生卒采用**通说**，不同史料和教材在早期年代（尤其夏商周断代）上存在差异。
- **中外对照仅表示大致同时期**，不代表事件之间存在因果关系。真正存在影响关系的节点已单独用「中外联动」卡片标出。
- 涉及近现代史的部分，表述以中国现行中学历史教材的通行口径为基础，旨在便于学生对照记忆。
- 发现史实错误请[提 Issue](https://github.com/yansheng836/parallel-histories/issues/new?template=fact_correction.yml)，**欢迎指正**。

## 关于这个项目的来源

页面初版内容由 AI 生成，经人工整理后开源。因此特别欢迎历史专业背景的贡献者审校——AI 生成的历史内容在细节、年代和人物关系上可能存在错误，请以权威史料为准。

## 许可证

[MIT License](LICENSE) — 可自由使用、修改、分发，包括商业用途，保留版权声明即可。

历史事实本身属于公共领域；本项目的排版、组织和文字表述采用 MIT 许可。

## 致谢

- 所有补充史实、修正错误的贡献者
- 以及每一位在历史教科书上留下笔记的读者

<div align="center">

**如果这个项目对你有帮助，欢迎点一个 ⭐ Star**

</div>
