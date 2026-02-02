# デザインシステム

CSS変数、テーマ定義、HTMLテンプレートの詳細。

---

## 必須ライブラリとフォント

```html
<!-- Fonts & Icons -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=BIZ+UDPGothic:wght@400;700&family=M+PLUS+Rounded+1c:wght@400;500;700&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">

<!-- Driver.js -->
<script src="https://cdn.jsdelivr.net/npm/driver.js@1.0.1/dist/driver.js.iife.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/driver.js@1.0.1/dist/driver.css"/>
```

### フォント優先順位

1. **Noto Sans JP** (メイン・本文用) - 最優先
2. BIZ UDPGothic (数字・データ用)
3. M PLUS Rounded 1c (アクセント用)

---

## CSS変数定義

### システム変数（共通）

```css
:root {
  /* Typography */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;

  --font-family: 'Noto Sans JP', 'BIZ UDPGothic', sans-serif;

  /* Spacing System (8px grid) */
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;

  /* Border Radius */
  --radius-sm: 6px;
  --radius-md: 12px;
  --radius-lg: 16px;

  /* Transitions */
  --transition-fast: 150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-base: 250ms cubic-bezier(0.4, 0, 0.2, 1);
}
```

---

## カラーシステム（12色構成 × 6テーマ）

各テーマで以下の12色を定義：

| 変数名 | 用途 |
|--------|------|
| --bg-base | 背景（基本） |
| --bg-surface | 背景（カード等） |
| --text-primary | テキスト（主） |
| --text-secondary | テキスト（副） |
| --text-inverse | テキスト（反転） |
| --border | ボーダー |
| --accent | アクセント |
| --accent-hover | アクセント（ホバー） |
| --shadow | シャドウ |
| --color-success | 成功 |
| --color-warning | 警告 |
| --color-danger | 危険 |

### ライト（デフォルト）

```css
:root {
  --bg-base: #f8fafc;
  --bg-surface: #ffffff;
  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --text-inverse: #ffffff;
  --border: rgba(15, 23, 42, 0.1);
  --accent: #4f46e5;
  --accent-hover: #4338ca;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;
}
```

### ダーク

```css
:root[data-theme="dark"] {
  --bg-base: #1e293b;
  --bg-surface: #334155;
  --text-primary: #f1f5f9;
  --text-secondary: #94a3b8;
  --text-inverse: #0f172a;
  --border: rgba(148, 163, 184, 0.2);
  --accent: #6366f1;
  --accent-hover: #818cf8;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  --color-success: #34d399;
  --color-warning: #fbbf24;
  --color-danger: #f87171;
}
```

### オーシャン

```css
:root[data-theme="ocean"] {
  --bg-base: #0c4a6e;
  --bg-surface: #075985;
  --text-primary: #e0f2fe;
  --text-secondary: #7dd3fc;
  --text-inverse: #0c4a6e;
  --border: rgba(125, 211, 252, 0.2);
  --accent: #0284c7;
  --accent-hover: #0ea5e9;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  --color-success: #2dd4bf;
  --color-warning: #fbbf24;
  --color-danger: #fb7185;
}
```

### フォレスト

```css
:root[data-theme="forest"] {
  --bg-base: #14532d;
  --bg-surface: #166534;
  --text-primary: #ecfdf5;
  --text-secondary: #6ee7b7;
  --text-inverse: #14532d;
  --border: rgba(110, 231, 183, 0.2);
  --accent: #10b981;
  --accent-hover: #34d399;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  --color-success: #22c55e;
  --color-warning: #fbbf24;
  --color-danger: #fb7185;
}
```

### サンセット

```css
:root[data-theme="sunset"] {
  --bg-base: #431407;
  --bg-surface: #7c2d12;
  --text-primary: #fff7ed;
  --text-secondary: #fdba74;
  --text-inverse: #431407;
  --border: rgba(253, 186, 116, 0.2);
  --accent: #f97316;
  --accent-hover: #fb923c;
  --shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  --color-success: #4ade80;
  --color-warning: #fbbf24;
  --color-danger: #f87171;
}
```

### サクラ

```css
:root[data-theme="sakura"] {
  --bg-base: #fdf2f8;
  --bg-surface: #ffffff;
  --text-primary: #831843;
  --text-secondary: #be185d;
  --text-inverse: #ffffff;
  --border: rgba(190, 24, 93, 0.15);
  --accent: #ec4899;
  --accent-hover: #f472b6;
  --shadow: 0 2px 8px rgba(236, 72, 153, 0.15);
  --color-success: #22c55e;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;
}
```

---

## 基本スタイル

```css
body {
  font-family: var(--font-family);
  background: var(--bg-base);
  color: var(--text-primary);
  transition: background 0.3s ease, color 0.3s ease;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: var(--space-4);
}
```

---

## ロード画面

```css
#app-loading {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: var(--bg-base);
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: opacity 0.8s ease, visibility 0.8s ease;
}

#app-loading.hidden {
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
}

.loading-logo {
  font-size: 4rem;
  margin-bottom: var(--space-4);
  color: var(--accent);
  animation: pulse-logo 2s infinite ease-in-out;
}

@keyframes pulse-logo {
  0%, 100% { opacity: 0.8; transform: scale(0.95); }
  50% { opacity: 1; transform: scale(1.05); }
}
```

---

## テーマ選択UI

```css
.theme-selector {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--space-4);
}

.theme-option {
  padding: var(--space-4);
  border-radius: var(--radius-md);
  cursor: pointer;
  border: 2px solid transparent;
  text-align: center;
  font-weight: 600;
  transition: var(--transition-fast);
}

.theme-option:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow);
}

.theme-option.selected {
  border-color: var(--accent);
  box-shadow: 0 0 0 2px var(--accent);
}

/* 各テーマボタンの固定色 */
.theme-option.light { background: #f8fafc; color: #0f172a; border: 1px solid #cbd5e1; }
.theme-option.dark { background: #1e293b; color: #f1f5f9; }
.theme-option.ocean { background: #0c4a6e; color: #e0f2fe; }
.theme-option.forest { background: #14532d; color: #ecfdf5; }
.theme-option.sunset { background: #431407; color: #fff7ed; }
.theme-option.sakura { background: #fdf2f8; color: #831843; border: 1px solid rgba(190, 24, 93, 0.2); }
```

---

## モーダル

```css
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.modal-content {
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  max-width: 600px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
}
```

---

## HTMLテンプレート

```html
<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- テーマ即時適用（白フラッシュ防止） -->
  <script>
    (function() {
      const savedTheme = localStorage.getItem('appTheme') || 'light';
      document.documentElement.setAttribute('data-theme', savedTheme);
    })();
  </script>

  <!-- Fonts & Icons -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=BIZ+UDPGothic:wght@400;700&family=M+PLUS+Rounded+1c:wght@400;500;700&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">

  <!-- Driver.js -->
  <script src="https://cdn.jsdelivr.net/npm/driver.js@1.0.1/dist/driver.js.iife.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/driver.js@1.0.1/dist/driver.css"/>

  <style>
    /* CSS変数とスタイルをここに */
  </style>
</head>
<body>
  <!-- ロード画面 -->
  <div id="app-loading">
    <div class="loading-logo">[アイコン]</div>
    <div class="loading-text">[アプリ名]</div>
  </div>

  <!-- メインコンテンツ -->
  <div class="container">
    <header class="header">
      <div class="header__brand">
        <div class="header__logo">[アイコン]</div>
        <div class="header__title">[アプリ名]</div>
      </div>
      <div class="header__actions">
        <button class="btn btn--icon" onclick="openSettingsModal()" title="設定">
          <span class="material-icons">settings</span>
        </button>
        <button class="btn btn--icon" onclick="startTour()" title="ヘルプ">
          <span class="material-icons">help_outline</span>
        </button>
      </div>
    </header>

    <main id="app-content">
      <!-- アプリ固有のコンテンツ -->
    </main>
  </div>

  <!-- 設定モーダル -->
  <div id="settingsModal" class="modal" style="display: none;">
    <div class="modal-content">
      <div class="modal-header">
        <h3>設定</h3>
        <button class="close-btn" onclick="closeModal('settingsModal')">&times;</button>
      </div>
      <div class="modal-body">
        <section class="settings-section">
          <h4>テーマ設定</h4>
          <div class="theme-selector" id="themeSelector"></div>
        </section>
        <section class="settings-section">
          <h4>フォント設定</h4>
          <select id="fontFamilySelect">
            <option value="'Noto Sans JP', sans-serif">Noto Sans JP (標準)</option>
            <option value="'BIZ UDPGothic', sans-serif">BIZ UDPGothic</option>
            <option value="'M PLUS Rounded 1c', sans-serif">M PLUS Rounded 1c</option>
          </select>
        </section>
        <div class="modal-actions">
          <button class="btn btn--ghost" onclick="resetSettings()">初期化</button>
          <button class="btn btn--primary" onclick="closeModal('settingsModal')">閉じる</button>
        </div>
      </div>
    </div>
  </div>

  <footer>
    <div class="footer-content">
      &copy; 2024 [アプリ名] | Developed by [Author]
    </div>
  </footer>

  <script>
    // JavaScriptをここに
  </script>
</body>
</html>
```

---

## 必須JavaScript関数

```javascript
// テーマ切り替え
function setTheme(theme) {
  document.documentElement.setAttribute('data-theme', theme);
  localStorage.setItem('appTheme', theme);
  updateThemeSelector();
}

// テーマセレクター生成
function generateThemeSelector() {
  const themes = [
    { id: 'light', name: 'ライト' },
    { id: 'dark', name: 'ダーク' },
    { id: 'ocean', name: 'オーシャン' },
    { id: 'forest', name: 'フォレスト' },
    { id: 'sunset', name: 'サンセット' },
    { id: 'sakura', name: 'サクラ' }
  ];

  const container = document.getElementById('themeSelector');
  container.innerHTML = themes.map(t =>
    `<div class="theme-option ${t.id}" onclick="setTheme('${t.id}')">${t.name}</div>`
  ).join('');
}

// モーダル操作
function openSettingsModal() {
  document.getElementById('settingsModal').style.display = 'flex';
}

function closeModal(id) {
  document.getElementById(id).style.display = 'none';
}

// モーダル外クリックで閉じる
window.onclick = function(event) {
  if (event.target.classList.contains('modal')) {
    event.target.style.display = 'none';
  }
};

// ツアー機能
function startTour() {
  const driver = window.driver.js.driver;
  const driverObj = driver({
    showProgress: true,
    allowKeyboardControl: true,
    popoverClass: 'driverjs-theme',
    steps: [
      { element: '.header__title', popover: { title: 'アプリ名', description: 'このアプリへようこそ！' }},
      { element: '#app-content', popover: { title: 'メインエリア', description: 'ここでメインの操作を行います。' }},
      { element: '[title="設定"]', popover: { title: '設定', description: 'テーマやフォントを変更できます。' }},
      { element: '[title="ヘルプ"]', popover: { title: 'ヘルプ', description: 'このツアーを再表示できます。' }},
      { popover: { title: '準備完了！', description: '早速使い始めましょう。' }}
    ]
  });
  driverObj.drive();
}

// 初期化
document.addEventListener('DOMContentLoaded', () => {
  generateThemeSelector();

  // ロード画面を非表示
  setTimeout(() => {
    document.getElementById('app-loading').classList.add('hidden');
  }, 500);

  // 初回訪問時はツアー開始
  if (localStorage.getItem('tourCompleted') !== 'true') {
    setTimeout(startTour, 1000);
    localStorage.setItem('tourCompleted', 'true');
  }
});
```
