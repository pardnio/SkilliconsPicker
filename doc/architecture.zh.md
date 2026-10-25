# SkilliconsPicker - 架構

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    U[使用者] --> P[index.html 頁面]
    P --> Q[QuickUI 0.6.5 執行環境]
    P --> L[圖示清單 Worker]
    P --> I[static/i18n JSON]
    P --> S[static/css/index.css]
    Q --> G["圖示牆 #picker-icon"]
    G --> R["結果區 #picker-result"]
    R --> O[skillicons.dev 圖示連結]
    O --> C[剪貼簿]
    O --> T[新分頁]
```

## Module: 初始化

`DOMContentLoaded` 時取得圖示清單、判斷語系並建立 QuickUI 實例。

```mermaid
graph TB
    subgraph 初始化
        A[DOMContentLoaded] --> B[fetch 圖示清單]
        B --> C{請求成功}
        C -->|是| D[icons = JSON 陣列]
        C -->|否| E[console.error，icons 為空]
        D --> F[讀取 ?lang 參數]
        E --> F
        F --> G{lang 存在}
        G -->|是| H[以 lang 判斷 zh]
        G -->|否| J[以 navigator.language 判斷 zh]
        H --> K[new QUI]
        J --> K
    end
    W[skillicon-list.pardn.workers.dev] --> B
    K --> N[i18n: zh.json / en.json]
    K --> M[data: lang、change、icons]
```

## Module: 圖示選擇

圖示牆以 `:for` 渲染每個圖示，點擊時切換 `data-selected` 並同步結果區。

```mermaid
graph TB
    subgraph 圖示選擇
        A[點擊圖示牆項目] --> B[切換 data-selected]
        B --> C{data-selected}
        C -->|1| D[建立 img 並綁定拖放與點擊事件]
        D --> E[append 至結果區]
        C -->|0| F[移除結果區對應 img]
        E --> G[change]
        F --> G
    end
    G --> H[更新 data-result]
```

## Module: 結果區操作

結果區負責排序、移除與輸出。

```mermaid
graph TB
    subgraph 結果區
        A[dragStart] --> B[dragOver]
        B --> C[drop]
        C --> D{拖曳方向}
        D -->|往前| E[insertBefore 目標]
        D -->|往後| F[insertBefore 目標下一個]
        E --> G[change]
        F --> G
        H[click / touchstart] --> I[removeFromResult]
        I --> J{clickCount}
        J -->|1| K[500 ms 計時重置]
        J -->|2| L[移除 img 並取消圖示牆勾選]
        L --> G
        G --> M[data-result = 依序 title 以逗號串接]
    end
    M --> N[copy: navigator.clipboard.writeText]
    M --> O[open: window.open]
    N --> P[按鈕文字 500 ms 顯示已拷貝]
```

## Module: 樣式與資源

```mermaid
graph TB
    subgraph 樣式
        A[function/_hsbTohsl.scss] --> C[index.scss]
        B[_init.scss 變數] --> C
        C -->|sass 編譯| D[static/css/index.css]
    end
    subgraph 外部資源
        E[jsDelivr: QuickUI、Font Awesome]
        F[Google Fonts: Material Symbols]
        G[Google Analytics gtag]
    end
    D --> H[index.html]
    E --> H
    F --> H
    G --> H
```

## 資料流

```mermaid
sequenceDiagram
    participant 使用者
    participant 頁面 as index.html
    participant Worker as 圖示清單 Worker
    participant QUI as QuickUI
    participant SI as skillicons.dev
    使用者->>頁面: 開啟頁面
    頁面->>Worker: GET /
    Worker-->>頁面: 圖示 ID 陣列
    頁面->>QUI: new QUI(i18n, data, event)
    QUI->>SI: 逐一載入圖示 icons?i={id}
    使用者->>QUI: 點擊圖示
    QUI->>頁面: 結果區新增 img、更新 data-result
    使用者->>頁面: 拖放排序／雙擊移除
    頁面->>頁面: change 重算 data-result
    使用者->>頁面: 拷貝連結
    頁面->>使用者: 剪貼簿寫入 icons?i={id1},{id2},...
```

## 狀態機

圖示牆單一項目的選取狀態：

```mermaid
stateDiagram-v2
    [*] --> 未選取
    未選取 --> 已選取: 點擊圖示牆
    已選取 --> 未選取: 再次點擊圖示牆
    已選取 --> 未選取: 結果區雙擊移除
```

結果區的雙擊判定：

```mermaid
stateDiagram-v2
    [*] --> 待機
    待機 --> 第一次點擊: click（clickCount = 1）
    第一次點擊 --> 待機: 500 ms 逾時重置
    第一次點擊 --> 移除: 500 ms 內第二次 click（clickCount = 2）
    移除 --> 待機: 清除計時並重置 clickCount
```

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
