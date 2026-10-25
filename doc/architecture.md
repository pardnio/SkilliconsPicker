# SkilliconsPicker - Architecture

Last updated: 2026-10-25

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    U[User] --> P[index.html Page]
    P --> Q[QuickUI 0.6.5 Runtime]
    P --> L[Icon List Worker]
    P --> I[static/i18n JSON]
    P --> S[static/css/index.css]
    Q --> G["Icon Wall #picker-icon"]
    G --> R["Result Panel #picker-result"]
    R --> O[skillicons.dev Icon URL]
    O --> C[Clipboard]
    O --> T[New Tab]
```

## Module: Bootstrap

On `DOMContentLoaded`, the page fetches the icon list, resolves the language, and creates the QuickUI instance.

```mermaid
graph TB
    subgraph Bootstrap
        A[DOMContentLoaded] --> B[fetch icon list]
        B --> C{Request OK}
        C -->|Yes| D[icons = JSON array]
        C -->|No| E[console.error, icons empty]
        D --> F[Read ?lang param]
        E --> F
        F --> G{lang present}
        G -->|Yes| H[Detect zh from lang]
        G -->|No| J[Detect zh from navigator.language]
        H --> K[new QUI]
        J --> K
    end
    W[skillicon-list.pardn.workers.dev] --> B
    K --> N[i18n: zh.json / en.json]
    K --> M[data: lang, change, icons]
```

## Module: Icon Selection

The icon wall renders each icon via `:for`; a click toggles `data-selected` and syncs the result panel.

```mermaid
graph TB
    subgraph Icon Selection
        A[Click icon wall item] --> B[Toggle data-selected]
        B --> C{data-selected}
        C -->|1| D[Create img and bind drag and click events]
        D --> E[Append to result panel]
        C -->|0| F[Remove matching img from result panel]
        E --> G[change]
        F --> G
    end
    G --> H[Update data-result]
```

## Module: Result Panel

The result panel handles ordering, removal, and output.

```mermaid
graph TB
    subgraph Result Panel
        A[dragStart] --> B[dragOver]
        B --> C[drop]
        C --> D{Drag direction}
        D -->|Backward| E[insertBefore target]
        D -->|Forward| F[insertBefore target next sibling]
        E --> G[change]
        F --> G
        H[click / touchstart] --> I[removeFromResult]
        I --> J{clickCount}
        J -->|1| K[Reset after 500 ms]
        J -->|2| L[Remove img and uncheck icon wall item]
        L --> G
        G --> M[data-result = titles joined by comma in order]
    end
    M --> N[copy: navigator.clipboard.writeText]
    M --> O[open: window.open]
    N --> P[Button shows Copied for 500 ms]
```

## Module: Styles and Assets

```mermaid
graph TB
    subgraph Styles
        A[function/_hsbTohsl.scss] --> C[index.scss]
        B[_init.scss variables] --> C
        C -->|sass compile| D[static/css/index.css]
    end
    subgraph External Assets
        E[jsDelivr: QuickUI, Font Awesome]
        F[Google Fonts: Material Symbols]
        G[Google Analytics gtag]
    end
    D --> H[index.html]
    E --> H
    F --> H
    G --> H
```

## Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Page as index.html
    participant Worker as Icon List Worker
    participant QUI as QuickUI
    participant SI as skillicons.dev
    User->>Page: Open page
    Page->>Worker: GET /
    Worker-->>Page: Icon ID array
    Page->>QUI: new QUI(i18n, data, event)
    QUI->>SI: Load each icon icons?i={id}
    User->>QUI: Click icon
    QUI->>Page: Add img to result panel, update data-result
    User->>Page: Drag to reorder / double-click to remove
    Page->>Page: change recomputes data-result
    User->>Page: Copy link
    Page->>User: Clipboard gets icons?i={id1},{id2},...
```

## State Machine

Selection state of a single icon wall item:

```mermaid
stateDiagram-v2
    [*] --> Unselected
    Unselected --> Selected: Click on icon wall
    Selected --> Unselected: Click again on icon wall
    Selected --> Unselected: Double-click in result panel
```

Double-click detection in the result panel:

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> FirstClick: click (clickCount = 1)
    FirstClick --> Idle: Reset after 500 ms
    FirstClick --> Removed: Second click within 500 ms (clickCount = 2)
    Removed --> Idle: Clear timer and reset clickCount
```

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
