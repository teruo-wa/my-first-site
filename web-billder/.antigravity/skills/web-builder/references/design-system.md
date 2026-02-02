# デザインシステム

## カラーパレット

### ライトモード
```css
--background: #ffffff;
--foreground: #171717;
--card: #f5f5f5;
--card-foreground: #171717;
--primary: #2563eb;
--primary-foreground: #ffffff;
--secondary: #f3f4f6;
--muted: #6b7280;
--border: #e5e7eb;
```

### ダークモード
```css
--background: #0a0a0a;
--foreground: #ededed;
--card: #171717;
--card-foreground: #ededed;
--primary: #3b82f6;
--primary-foreground: #ffffff;
--secondary: #1f2937;
--muted: #9ca3af;
--border: #374151;
```

## タイポグラフィ

### フォントファミリー
```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

### フォントサイズ
- 見出し1: 3rem (48px) / font-weight: 700
- 見出し2: 2.25rem (36px) / font-weight: 600
- 見出し3: 1.5rem (24px) / font-weight: 600
- 本文: 1rem (16px) / font-weight: 400
- 小文字: 0.875rem (14px) / font-weight: 400

## スペーシング

Tailwind CSSのスペーシングスケールを使用：
- xs: 0.25rem (4px)
- sm: 0.5rem (8px)
- md: 1rem (16px)
- lg: 1.5rem (24px)
- xl: 2rem (32px)
- 2xl: 3rem (48px)
- 3xl: 4rem (64px)

## ブレークポイント

```css
sm: 640px   /* モバイル横向き */
md: 768px   /* タブレット */
lg: 1024px  /* 小型デスクトップ */
xl: 1280px  /* デスクトップ */
2xl: 1536px /* 大型デスクトップ */
```

## シャドウ

```css
/* カード用 */
shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1), 0 2px 4px rgba(0, 0, 0, 0.06);
shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);

/* ダークモード用（より微妙なシャドウ） */
dark:shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
```

## 角丸

```css
rounded-sm: 0.125rem (2px)
rounded: 0.25rem (4px)
rounded-md: 0.375rem (6px)
rounded-lg: 0.5rem (8px)
rounded-xl: 0.75rem (12px)
rounded-2xl: 1rem (16px)
rounded-3xl: 1.5rem (24px)
```

## トランジション

```css
/* 基本トランジション */
transition-all duration-200 ease-out

/* ホバー効果用 */
transition-all duration-300 ease-out

/* ページ遷移用 */
transition-all duration-500 cubic-bezier(0.87, 0, 0.13, 1)
```

## カード設計パターン

```tsx
// 基本カード
<div className="p-6 bg-white dark:bg-neutral-800 rounded-xl border border-neutral-200 dark:border-neutral-700 shadow-sm hover:shadow-md transition-shadow">
  {/* content */}
</div>

// インタラクティブカード（クリック可能）
<div className="p-6 bg-white dark:bg-neutral-800 rounded-xl border border-neutral-200 dark:border-neutral-700 shadow-sm hover:shadow-md hover:border-primary-400 transition-all cursor-pointer">
  {/* content */}
</div>

// グラデーションカード
<div className="p-6 bg-gradient-to-br from-primary-500 to-primary-700 rounded-xl text-white">
  {/* content */}
</div>
```
