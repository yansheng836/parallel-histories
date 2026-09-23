<div align="center">

# Parallel Histories

**A Side-by-Side Timeline of Chinese and World History**

Read Chinese history and world history on the same timeline.

[![Live Demo](https://img.shields.io/badge/Live%20Demo-yansheng836.github.io-2c3e50?style=flat-square)](https://yansheng836.github.io/parallel-histories/)
[![License: MIT](https://img.shields.io/badge/License-MIT-c0392b?style=flat-square)](LICENSE)
[![Single File](https://img.shields.io/badge/Single%20File-HTML-2471a3?style=flat-square)](#)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-27ae60?style=flat-square)](#)

**English** | [简体中文](README.md)

<!-- TODO: replace with a real screenshot
     1. Open index.html locally (or the live version)
     2. Set the window to 1280px wide and capture the Sui-Tang through Ming-Qing section
        (the side-by-side layout reads best there)
     3. Save as docs/preview.png, then delete this comment and the notice line below
-->
> 📷 *Preview image coming soon — see the [live demo](https://yansheng836.github.io/parallel-histories/) meanwhile*

</div>

---

## What is this

A **single-file, zero-dependency** static web page. A vertical timeline runs down the center: **Chinese events on the left, Western / world events on the right**, so events from the same period sit side by side for comparison.

It spans from **c. 2070 BCE (the founding of the Xia dynasty)** all the way to the **present day**, covering:

| Period | What is compared |
|--------|------------------|
| Ancient | Xia / Shang / Zhou ←→ Aegean civilization, Greek city-states, Alexander's conquests |
| Qin–Han | The first unified empire ←→ Roman Republic and Roman Empire |
| Three Kingdoms to Northern & Southern Dynasties | Ethnic fusion ←→ Fall of Western Rome, start of the European Middle Ages |
| Sui–Tang | The Sui–Tang golden age ←→ Byzantium, rise of the Arab Empire |
| Song–Yuan | Peak of economy and culture ←→ Crusades, Magna Carta, Marco Polo's journey east |
| Ming–Qing | Absolutism and seclusion ←→ Renaissance, Age of Discovery, Reformation, Industrial Revolution |
| Late Qing | Opium War to the Boxer Protocol ←→ Spread of capitalism, Second Industrial Revolution |
| Republic of China | 1911 Revolution to victory in WWII ←→ Both World Wars, October Revolution |
| Contemporary | PRC to the present ←→ Cold War, globalization, multipolarity |

A special set of **"Cross-Civilization Links"** cards (dashed purple boxes) marks the moments when the two civilizations genuinely interacted, for example:

- Zhang Qian's mission to the Western Regions opens the **Silk Road** — Chinese silk reaches Rome
- Industrial Revolution → opium smuggling → destruction of opium at Humen → **Opium War**
- Paris Peace Conference → **May Fourth Movement**; October Revolution → founding of the CCP
- 1971 UN seat restored → 1972 Nixon visits China → 1979 US–China diplomatic relations

## Features

- **Zero dependencies** — no npm, no build step, no CDN. One `index.html` is the whole thing
- **Works offline** — download it and double-click; no network required
- **Responsive** — side-by-side layout on desktop; switches to a single-column timeline on mobile (≤768px)
- **Scroll animations** — native `IntersectionObserver` fades cards in as they enter the viewport
- **Period bands** — era color bands automatically align to timeline ticks, kept in sync via `ResizeObserver`
- **Print-friendly** — printing gives you a usable China–world comparative chronology

## Getting started

### Online

<https://yansheng836.github.io/parallel-histories/>

### Run locally

No toolchain needed. Pick any of these:

```bash
# Option 1: open it directly
start index.html        # Windows
open index.html         # macOS

# Option 2: serve locally (recommended for testing on a phone over LAN)
python -m http.server 8000
# then open http://localhost:8000

# Option 3: VS Code + Live Server extension → right-click index.html → Open with Live Server
```

## Contributing

Content additions are very welcome. This project **only requires editing HTML** — there is no build step.

### Add a historical event

1. Open `index.html`
2. Find the matching `.row` block (`Ctrl+F` for the year, e.g. `1840年`)
3. Put the Chinese event in `.card.cn` and the contemporaneous Western event in `.card.west`

```html
<div class="row">
  <div class="card cn"><i class="node"></i>
    <h3>Event title <span class="y">(years)</span></h3>
    <div class="fig">Key figures and details</div>
    One sentence on why it mattered.</div>
  <div class="card west"><i class="node"></i>
    <h3>Contemporaneous Western event <span class="y">(year)</span></h3>
    Explanation.</div>
</div>
```

### Add a timeline tick

```html
<div class="tick" id="t-year"><span class="year">Year</span><span class="era-tag l">Label</span></div>
```

### Add a cross-civilization link

```html
<div class="link-note"><div class="inner">
  ⚡ <b>Theme</b>: cause → effect → consequence.
</div></div>
```

### Style conventions

| Class | Purpose |
|-------|---------|
| `.card.cn` | Left-hand Chinese card (red) |
| `.card.west` | Right-hand Western card (blue) |
| `.card .y` | Year inside a card title, rendered dimmed |
| `.card .fig` | Key figures / details line |
| `.link-note` | Cross-civilization link card (dashed purple) |
| `.era-cn` / `.era-west` | Era bands on either side; `data-from` / `data-to` pick the ticks |

When adding an era band, `data-from` / `data-to` must reference existing tick `id`s — the script computes position and height automatically.

### Open a pull request

```bash
git clone git@github.com:yansheng836/parallel-histories.git
cd parallel-histories
git checkout -b feat/add-ming-dynasty-detail
# edit index.html
git commit -m "feat: 补充明朝郑和下西洋细节"
git push -u origin feat/add-ming-dynasty-detail
```

Commit messages follow `type: description`. This project is Chinese-first, so Chinese commit descriptions are the norm.

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Scope and disclaimers

- This is a **study aid**, not a scholarly work. Years and biographical dates follow **mainstream consensus**; sources and textbooks differ on early dates (especially the Xia–Shang–Zhou chronology).
- **Side-by-side placement means roughly contemporaneous only** — it does not imply causation. Points of genuine mutual influence are called out separately in the "Cross-Civilization Links" cards.
- Sections on modern history follow the standard narrative of current Chinese secondary-school history textbooks, so that students can use it for comparative review.
- Spotted a factual error? Please [open an issue](https://github.com/yansheng836/parallel-histories/issues/new?template=fact_correction.yml) — **corrections are welcome**.

## Origin of this project

The initial page content was AI-generated and then organized for release. Contributions from people with a history background are especially welcome: AI-generated historical content can contain errors in detail, dating, and personal relationships. Authoritative sources take precedence.

## License

[MIT License](LICENSE) — free to use, modify, and distribute, including commercially, provided the copyright notice is retained.

Historical facts are themselves in the public domain; the layout, organization, and prose of this project are released under MIT.

## Acknowledgements

- Everyone who has added facts or corrected errors
- And every reader who has ever scribbled notes in a history textbook

<div align="center">

**If this project helps you, a ⭐ Star is appreciated**

</div>
