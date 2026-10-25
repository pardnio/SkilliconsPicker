# SkilliconsPicker - Documentation

Last updated: 2026-10-25

> Back to [README](../README.md)

## Prerequisites

- A modern browser with ES2017 `async`/`await`, Fetch API, Clipboard API, and HTML5 Drag and Drop
- Network access to the following services:
  - `https://skillicon-list.pardn.workers.dev/`: icon list source
  - `https://skillicons.dev`: icon images and output URL
  - `https://cdn.jsdelivr.net`: `@pardnchiu/quickui@0.6.5` and Font Awesome 6.5.2
  - `https://fonts.googleapis.com`: Material Symbols Outlined icon font
- Any static file server (the page loads `static/i18n/*.json` via `fetch`, which fails over `file://`)
- [Sass](https://sass-lang.com/) (Dart Sass) only when editing styles

## Installation

### Run from Source

```bash
git clone https://github.com/pardnchiu/SkilliconsPicker.git
cd SkilliconsPicker
python3 -m http.server 8080
```

Open `http://localhost:8080/` in a browser.

### Rebuild Styles (Optional)

`static/css/index.css` is compiled from `static/sass/index.scss`. Recompile after editing SCSS:

```bash
sass static/sass/index.scss static/css/index.css
```

### Hosted Version

GitHub Pages: <https://pardnio.github.io/SkilliconsPicker/>

## Usage

### Basic

1. Open the page; the "All Icons" section lists every icon from the remote list
2. Click an icon to add it to "Icons you picked"; click again to remove it
3. Press "Copy link" to write the result URL to the clipboard, or "Open Link" to preview the image in a new tab

Generated URL format:

```text
https://skillicons.dev/icons?i=js,ts,react,go
```

Use it in Markdown:

```markdown
![Skills](https://skillicons.dev/icons?i=js,ts,react,go)
```

### Reorder and Remove

- Drag an icon in the result panel onto another icon to reorder; the output URL follows the left-to-right order of the result panel
- Double-click an icon in the result panel (two clicks within 500 ms) to remove it; its checked state on the icon wall clears as well

### Switch Language

```text
http://localhost:8080/index.html?lang=en
http://localhost:8080/index.html?lang=zh
```

Without `lang`, the page checks `navigator.language`: values starting with `zh` use Chinese, everything else uses English. The language link at the bottom-right corner switches to the other language.

## Configuration Reference

### URL Parameters

| Parameter | Values | Default | Description |
|-----------|--------|---------|-------------|
| `lang` | Starts with `zh` (case-insensitive) → Chinese; any other value → English | From `navigator.language` | Overrides browser language detection |

### i18n Keys

`static/i18n/zh.json` and `static/i18n/en.json` share the same structure:

| Key | zh | en | Purpose |
|-----|----|----|---------|
| `title` | 使用 Skill Icons 圖示所製作的選擇器 | Picker made using Skill Icons. | Page title (contains an HTML link) |
| `picked` | 你的選擇 | Icons you picked | Result panel heading |
| `drag` | 拖曳調整順序 | Drag to adjust order | Drag hint |
| `tap` | 雙擊移除圖示 | Double click to remove | Removal hint |
| `empty` | 尚未選擇圖示 | No icons selected yet | Empty result placeholder |
| `copy` | 拷貝連結 | Copy link | Copy button |
| `open` | 開啟連結 | Open Link | Open button |
| `all` | 所有圖示 | All Icons | Icon wall heading |

After copying, the button text swaps `拷貝連結`/`Copy link` for `已拷貝`/`Copied` by string matching and restores it after 500 ms; when changing the `copy` translation, update the matching strings in `index.html` too.

### External Endpoints

| Endpoint | Response | Purpose |
|----------|----------|---------|
| `GET https://skillicon-list.pardn.workers.dev/` | JSON array of icon IDs (e.g. `["ableton","activitypub",...]`) | Icon wall data source; on failure it only calls `console.error` and the icon wall stays empty |
| `https://skillicons.dev/icons?i={id}` | SVG image | Single icon preview |
| `https://skillicons.dev/icons?i={id1},{id2},...` | SVG image | Final output URL |

### Project Files

| Path | Description |
|------|-------------|
| `index.html` | Page markup, QuickUI instance, and all interaction logic |
| `static/i18n/*.json` | Interface strings |
| `static/sass/` | Style sources (`_init.scss` variables, `function/_hsbTohsl.scss` provides `hsb()`/`hsba()`) |
| `static/css/index.css` | Compiled styles loaded by the page |
| `static/image/logo.svg` | Header logo |
| `icons` | Newline-separated local snapshot of the icon list; `index.html` does not read it currently |

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
