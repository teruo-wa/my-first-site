# アニメーション実装パターン

## ページ遷移アニメーション

### 概要
- **Reveal**: ページ読み込み時、オーバーレイが上にスライドして消える
- **Cover**: リンククリック時、オーバーレイが下から上にスライドして画面を覆う

### 実装例（Next.js App Router）

#### 1. PageTransition コンポーネント

```tsx
// components/PageTransition.tsx
"use client";

import { useEffect, useState } from "react";
import { usePathname } from "next/navigation";

export function PageTransition({ children }: { children: React.ReactNode }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const pathname = usePathname();

  useEffect(() => {
    // ページ読み込み完了後にRevealアニメーション
    const timer = setTimeout(() => setIsLoaded(true), 100);
    return () => clearTimeout(timer);
  }, [pathname]);

  return (
    <>
      {/* オーバーレイ */}
      <div
        className={`fixed inset-0 bg-neutral-900 dark:bg-neutral-100 z-50 transition-transform duration-500 origin-top ${
          isLoaded ? "scale-y-0" : "scale-y-100"
        }`}
        style={{ transitionTimingFunction: "cubic-bezier(0.87, 0, 0.13, 1)" }}
      />
      {children}
    </>
  );
}
```

#### 2. TransitionLink コンポーネント

```tsx
// components/TransitionLink.tsx
"use client";

import { useRouter } from "next/navigation";
import { useState } from "react";

interface TransitionLinkProps {
  href: string;
  children: React.ReactNode;
  className?: string;
}

export function TransitionLink({ href, children, className }: TransitionLinkProps) {
  const router = useRouter();
  const [isExiting, setIsExiting] = useState(false);

  const handleClick = (e: React.MouseEvent) => {
    e.preventDefault();
    setIsExiting(true);

    // Coverアニメーション後に遷移
    setTimeout(() => {
      router.push(href);
    }, 500);
  };

  return (
    <>
      {isExiting && (
        <div
          className="fixed inset-0 bg-neutral-900 dark:bg-neutral-100 z-50 animate-cover"
          style={{
            animation: "cover 0.5s cubic-bezier(0.87, 0, 0.13, 1) forwards",
          }}
        />
      )}
      <a href={href} onClick={handleClick} className={className}>
        {children}
      </a>
    </>
  );
}
```

#### 3. CSS アニメーション定義

```css
/* globals.css に追加 */
@keyframes cover {
  from {
    transform: scaleY(0);
    transform-origin: bottom;
  }
  to {
    transform: scaleY(1);
    transform-origin: bottom;
  }
}

@keyframes reveal {
  from {
    transform: scaleY(1);
    transform-origin: top;
  }
  to {
    transform: scaleY(0);
    transform-origin: top;
  }
}
```

## マイクロインタラクション

### ホバーエフェクト

```tsx
// カードのホバー
<div className="transform hover:scale-[1.02] hover:-translate-y-1 transition-all duration-300">
  {/* content */}
</div>

// ボタンのホバー
<button className="hover:brightness-110 active:scale-95 transition-all duration-200">
  Click me
</button>
```

### フェードイン（スクロール連動）

```tsx
// components/FadeIn.tsx
"use client";

import { useEffect, useRef, useState } from "react";

export function FadeIn({ children, delay = 0 }: { children: React.ReactNode; delay?: number }) {
  const ref = useRef<HTMLDivElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setTimeout(() => setIsVisible(true), delay);
        }
      },
      { threshold: 0.1 }
    );

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [delay]);

  return (
    <div
      ref={ref}
      className={`transition-all duration-700 ${
        isVisible ? "opacity-100 translate-y-0" : "opacity-0 translate-y-8"
      }`}
    >
      {children}
    </div>
  );
}
```

### ローディングスピナー

```tsx
// components/Spinner.tsx
export function Spinner({ size = "md" }: { size?: "sm" | "md" | "lg" }) {
  const sizeClasses = {
    sm: "w-4 h-4",
    md: "w-6 h-6",
    lg: "w-8 h-8",
  };

  return (
    <div
      className={`${sizeClasses[size]} border-2 border-neutral-300 border-t-primary-500 rounded-full animate-spin`}
    />
  );
}
```
