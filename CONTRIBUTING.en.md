# Contributing

Thanks for your interest in Parallel Histories. The project is deliberately minimal — **one `index.html` is the whole thing** — so contributing is straightforward.

**English** | [简体中文](CONTRIBUTING.md)

---

## Contents

- [What to contribute](#what-to-contribute)
- [Development setup](#development-setup)
- [Content conventions](#content-conventions)
- [HTML structure reference](#html-structure-reference)
- [Opening a pull request](#opening-a-pull-request)
- [Commit message format](#commit-message-format)
- [Accuracy requirements](#accuracy-requirements)
- [Code of conduct](#code-of-conduct)

---

## What to contribute

Ordered by value:

| Type | Description | Effort |
|------|-------------|--------|
| **Factual corrections** | Wrong year, figure, or description — the most valuable contribution | ⭐ |
| **Added detail** | Key figures, exact years, consequences for existing entries | ⭐ |
| **New events** | New timeline ticks or comparison entries | ⭐⭐ |
| **New cross-civilization links** | Find moments where the two civilizations genuinely met | ⭐⭐ |
| **Styling / interaction** | Responsiveness, accessibility, print styles, performance | ⭐⭐⭐ |
| **Translation** | Keeping the English version in sync, other languages | ⭐⭐ |

**Not** needed: build config (there is no build), new dependencies (zero-dependency is deliberate), frameworks (vanilla HTML/CSS/JS is a design decision, not technical debt).

Note that the page content itself is Chinese-first. Translations of the *page content* are a larger discussion — [open an issue](https://github.com/yansheng836/parallel-histories/issues) before starting one.

## Development setup

None required.

```bash
git clone git@github.com:yansheng836/parallel-histories.git
cd parallel-histories

# serve locally, then just refresh after each edit
python -m http.server 8000
```

Or open `index.html` directly in a browser. Save → refresh → see the result.

For checking mobile layout, use Chrome/Edge DevTools device emulation (`Ctrl+Shift+M`) at 375px width.

## Content conventions

Page content is written in Chinese. If you contribute content, follow these conventions.

### Year format

| Case | Format |
|------|--------|
| BCE | `前221年`, `约前2070年` |
| CE | `618年`, `1949年` |
| Range | `（618—907）` — em dash, no trailing 年 |
| Exact date | `1911.10.10`, `1949.10.1` |
| Century | `18世纪中叶` |
| Approximate | `约前2000—前1200`, `19世纪60—90年代` |

### Card structure

Each card has three parts, in order:

```html
<div class="card cn"><i class="node"></i>
  <h3>Event title <span class="y">(start–end years)</span></h3>
  <div class="fig">Key figures and details</div>
  One sentence on the historical significance.</div>
```

- The `<span class="y">` inside `<h3>` holds the year range and is rendered dimmed
- `<div class="fig">` holds **figures and details** (bold names with `<b>`)
- The final line is the **significance summary**, with no wrapper element
- First mention of a person uses `<b>汉武帝刘彻</b>（前141-87在位）`

### When to use a cross-civilization link

Use `link-note` **only when the two civilizations actually influenced each other** — not merely for "same period".

✅ Good examples:

- Zhang Qian → Silk Road → Chinese silk reaches Rome
- Battle of Talas (751) → papermaking spreads west
- Industrial Revolution → opium smuggling → Humen → Opium War
- October Revolution → Marxism reaches China → founding of the CCP

❌ Bad examples:

- "The Arab Empire was also rising during the Tang" — contemporaneous, no direct link
- "Europe was fighting the Crusades during the Song" — parallel, not linked

## HTML structure reference

### Timeline tick

```html
<div class="tick" id="t-tang"><span class="year">618年</span><span class="era-tag l">唐朝建立</span></div>
```

- `id` convention: `t-` + short slug (`t-tang`, `t-1840`, `t-sui`)
- `.era-tag.l` renders the label on the left, `.era-tag.r` on the right (desktop only; mobile inlines it)

### Comparison row

```html
<div class="row">
  <div class="card cn"><i class="node"></i>…</div>
  <div class="card west"><i class="node"></i>…</div>
</div>
```

One `.row` is one left-right comparison. If only one side has content, a single card is fine — the layout handles it. Multiple `.row`s can share one `.tick` for several events in the same period.

### Era band

```html
<div class="era-cn" data-from="#t-qin" data-to="#t-sanguo">秦汉·大一统</div>
<div class="era-west" data-from="#t-sui" data-to="#t-ming">中世纪·基督教文明</div>
```

- `data-to` may be omitted to extend to the end of the timeline
- `data-from` / `data-to` **must reference existing `id`s**, otherwise the script skips the band
- `.era-cn` hugs the left edge, `.era-west` the right

### Cross-civilization link

```html
<div class="link-note"><div class="inner">
  ⚡ <b>Theme</b>: cause → effect → consequence.
</div></div>
```

Placed between two `.row`s, as its own centered block.

## Opening a pull request

```bash
# 1. branch from up-to-date main
git checkout main && git pull
git checkout -b fix/tang-dynasty-date

# 2. edit index.html

# 3. test locally (important)
#    - Desktop: 1280px — cards aligned, era bands flush with ticks
#    - Mobile:  375px  — single column, card arrows correct
#    - DevTools Console: no errors

# 4. commit and push
git add index.html
git commit -m "fix: 修正唐朝建立年份"
git push -u origin fix/tang-dynasty-date

# 5. open the PR on GitHub and fill in .github/PULL_REQUEST_TEMPLATE.md
```

### PR checklist

- [ ] Only the necessary content changed — no reformatting the whole file (it makes the diff unreadable)
- [ ] Checked the layout at 1280px on desktop
- [ ] Checked the layout at 375px on mobile
- [ ] No console errors
- [ ] Any new `data-from` / `data-to` points at an existing `id`
- [ ] Facts are sourced, not written from memory (see below)
- [ ] README preview image updated if it is now stale (optional)

## Commit message format

Use `type: description`:

```text
feat: 补充明朝郑和下西洋细节
fix: 修正安史之乱结束年份
docs: 更新贡献指南的年份写法说明
style: 优化手机端卡片间距
refactor: 提取时间轴定位逻辑为独立函数
chore: 添加 GitHub Pages 部署工作流
```

| type | Use for |
|------|---------|
| `feat` | New events, ticks, link cards |
| `fix` | Factual errors, layout bugs |
| `docs` | Documentation only |
| `style` | Styling only, no content change |
| `refactor` | Code restructuring, no behavior change |
| `test` | Tests |
| `ci` | CI / deployment config |
| `chore` | Everything else |

Descriptions are written in Chinese to match the project. If you are not comfortable writing Chinese, an English description is fine — just keep the `type:` prefix.

## Accuracy requirements

This section matters most.

1. **Prefer mainstream consensus.** Where accounts differ, follow the prevailing view in standard textbooks and scholarship, and note the disagreement in the card if it is significant.
2. **Never write a year from memory.** If unsure, verify it or qualify it with 约 ("approximately").
3. **Distinguish "event occurred" from "regime established."** The Sui was founded in 581 and completed unification in 589 — not the same year.
4. **Watch the calendar conversion.** For early Chinese chronology (Xia–Shang–Zhou) this project follows the Xia–Shang–Zhou Chronology Project, which differs from some overseas scholarship.
5. **Keep modern history measured.** Wording follows current Chinese secondary-school history textbooks; avoid emotive language and evaluation beyond what the facts support.
6. **When birth/death dates are uncertain**, give reign dates instead, e.g. `（前141-87在位）`.

If you are submitting a **correction**, include your source in the PR description (textbook title, scholarly work, or reliable online source) — it speeds up review considerably.

## Code of conduct

- Discuss historical questions **academically**. Respect differing views; no political attacks or personal attacks.
- History is contentious. Where a phrasing is subject to long-running disagreement, maintainers may choose **neutral wording** or **note multiple interpretations**. That is not a political stance.
- Discriminatory language of any kind is not accepted.
- Violations will be closed; repeat violations will be blocked.

---

<div align="center">

Questions? [Open an issue](https://github.com/yansheng836/parallel-histories/issues) or just ask in your PR.

</div>
