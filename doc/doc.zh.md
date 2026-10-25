# SkilliconsPicker - 技術文件

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 前置需求

- 支援 ES2017 `async`／`await`、Fetch API、Clipboard API 與 HTML5 拖放（Drag and Drop）的現代瀏覽器
- 可連線至下列外部服務：
  - `https://skillicon-list.pardn.workers.dev/`：圖示清單來源
  - `https://skillicons.dev`：圖示圖片與輸出連結
  - `https://cdn.jsdelivr.net`：`@pardnchiu/quickui@0.6.5` 與 Font Awesome 6.5.2
  - `https://fonts.googleapis.com`：Material Symbols Outlined 圖示字型
- 任一靜態檔案伺服器（頁面以 `fetch` 讀取 `static/i18n/*.json`，`file://` 直接開啟無法載入）
- 僅在修改樣式時需要 [Sass](https://sass-lang.com/)（Dart Sass）

## 安裝

### 從原始碼執行

```bash
git clone https://github.com/pardnchiu/SkilliconsPicker.git
cd SkilliconsPicker
python3 -m http.server 8080
```

以瀏覽器開啟 `http://localhost:8080/`。

### 重新編譯樣式（選用）

`static/css/index.css` 由 `static/sass/index.scss` 編譯而來，修改 SCSS 後重新輸出：

```bash
sass static/sass/index.scss static/css/index.css
```

### 線上版本

GitHub Pages：<https://pardnio.github.io/SkilliconsPicker/>

## 使用方式

### 基礎用法

1. 開啟頁面後，「所有圖示」區塊會列出遠端清單中的全部圖示
2. 點擊圖示即加入「你的選擇」，再點一次則移出
3. 按「拷貝連結」將結果網址寫入剪貼簿，按「開啟連結」在新分頁預覽圖片

產生的網址格式：

```text
https://skillicons.dev/icons?i=js,ts,react,go
```

在 Markdown 中使用：

```markdown
![Skills](https://skillicons.dev/icons?i=js,ts,react,go)
```

### 調整順序與移除

- 在結果區拖曳圖示到另一個圖示上即可調整順序，輸出網址依結果區由左至右的順序組成
- 在結果區雙擊（500 ms 內點兩下）圖示即移除，圖示牆上的勾選狀態同步取消

### 切換語系

```text
http://localhost:8080/index.html?lang=en
http://localhost:8080/index.html?lang=zh
```

未帶 `lang` 時依 `navigator.language` 判斷：以 `zh` 開頭使用中文，其餘使用英文。頁面右下角的語系連結會切換到另一個語系。

## 設定參考

### URL 參數

| 參數 | 值 | 預設 | 說明 |
|------|----|------|------|
| `lang` | 以 `zh` 開頭（不分大小寫）→ 中文；其他任意值 → 英文 | 依 `navigator.language` | 覆寫瀏覽器語系偵測 |

### i18n 鍵值

`static/i18n/zh.json` 與 `static/i18n/en.json` 結構相同：

| 鍵 | zh | en | 用途 |
|----|----|----|------|
| `title` | 使用 Skill Icons 圖示所製作的選擇器 | Picker made using Skill Icons. | 頁面標題（含 HTML 連結） |
| `picked` | 你的選擇 | Icons you picked | 結果區標題 |
| `drag` | 拖曳調整順序 | Drag to adjust order | 拖曳提示 |
| `tap` | 雙擊移除圖示 | Double click to remove | 移除提示 |
| `empty` | 尚未選擇圖示 | No icons selected yet | 結果區空白提示 |
| `copy` | 拷貝連結 | Copy link | 複製按鈕 |
| `open` | 開啟連結 | Open Link | 開啟按鈕 |
| `all` | 所有圖示 | All Icons | 圖示牆標題 |

複製後按鈕文字會以 `拷貝連結`／`Copy link` 字串比對替換為 `已拷貝`／`Copied`，500 ms 後還原；修改 `copy` 的翻譯時需同步調整 `index.html` 內的比對字串。

### 外部端點

| 端點 | 回應 | 用途 |
|------|------|------|
| `GET https://skillicon-list.pardn.workers.dev/` | 圖示 ID 的 JSON 陣列（如 `["ableton","activitypub",...]`） | 圖示牆資料來源；請求失敗時僅 `console.error`，圖示牆為空 |
| `https://skillicons.dev/icons?i={id}` | SVG 圖片 | 單一圖示預覽 |
| `https://skillicons.dev/icons?i={id1},{id2},...` | SVG 圖片 | 最終輸出連結 |

### 專案檔案

| 路徑 | 說明 |
|------|------|
| `index.html` | 頁面結構、QuickUI 實例與所有互動邏輯 |
| `static/i18n/*.json` | 介面字串 |
| `static/sass/` | 樣式原始碼（`_init.scss` 變數、`function/_hsbTohsl.scss` 提供 `hsb()`／`hsba()`） |
| `static/css/index.css` | 編譯後樣式，由頁面直接載入 |
| `static/image/logo.svg` | 頁首 Logo |
| `icons` | 換行分隔的本地圖示清單快照；目前 `index.html` 未讀取此檔 |

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
