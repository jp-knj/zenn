---
title: "Islands Architectureの理論と実装"
---

# Islands Architectureの理論と実装

## TL;DR
- Jason Millerが提唱したIslands Architectureをコードレベルで実装したのがAstroの核心
- 静的HTML + 選択的ハイドレーションにより、パフォーマンスと開発体験を両立
- `<astro-island>`要素とメタデータ埋め込みにより、フレームワーク横断的な部分ハイドレーションを実現

---

## 1. 背景：SPAの限界と新しいパラダイム

### JavaScript配信の課題
2020年頃のWeb開発では、平均的なWebサイトが400KBのJavaScriptを配信していました。React、Vue、Angularの普及により、**すべてのページがJavaScriptアプリケーション**として構築される時代でしたが、これは根本的な問題を抱えていました：

- モバイルデバイスでの解析・実行コストの増大
- Time to Interactive（TTI）の悪化
- SEOとアクセシビリティの課題

### Islands Architectureの提唱
2020年8月、PreactのクリエイターJason Millerが革新的なアイデアを提唱しました：

> 「ページの大部分は静的なHTMLとして配信され、動的な領域は独立した埋め込みアプリケーションとして配置される」

---

## 2. コア概念：海と島のメタファー

### 静的な海（Static Ocean）
```html
<!-- 通常のHTML：JS実行なし、即座に表示 -->
<html>
<head><title>高速表示</title></head>
<body>
  <header>ナビゲーション</header>
  <main>記事コンテンツ</main>
  <footer>フッター情報</footer>
</body>
</html>
```

### インタラクティブな島（Interactive Islands）
```html
<!-- 必要な部分のみJSで拡張 -->
<astro-island 
  component-url="/_astro/SearchBox.js"
  renderer-url="/_astro/client-react.js"
  props='{"placeholder":"検索..."}'
  client:load>
  <div>検索フォーム（静的HTML）</div>
</astro-island>
```

---

## 3. 実装ウォークスルー：Astroの内部メカニズム

### ビルド時処理
```typescript
// packages/astro/src/core/build/island.ts
export function generateIsland(component, props, directive) {
  return {
    componentUrl: bundleComponent(component),
    rendererUrl: getRenderer(component.framework),
    staticHtml: renderToStaticMarkup(component, props),
    hydrationData: {
      directive,
      props: serializeProps(props)
    }
  }
}
```

### ランタイム処理
```typescript
// client.ts
class AstroIsland extends HTMLElement {
  async connectedCallback() {
    const strategy = this.getAttribute('client:load');
    const componentUrl = this.getAttribute('component-url');
    
    if (strategy === 'load') {
      await this.hydrate(componentUrl);
    }
  }
  
  async hydrate(url) {
    const { default: Component } = await import(url);
    const props = JSON.parse(this.getAttribute('props'));
    // フレームワーク固有のハイドレーション実行
  }
}
```

---

## 4. 他フレームワークとの比較

| アプローチ | Next.js | Nuxt.js | **Astro** |
|----------|---------|---------|-----------|
| 基本戦略 | すべてハイドレート | すべてハイドレート | **選択的ハイドレート** |
| JS初期サイズ | 100-300KB | 80-250KB | **10-50KB** |
| TTI | 3-5秒 | 2-4秒 | **1-2秒** |
| SEO | ○ | ○ | **◎** |

### Next.js App Router vs Astro Islands
```jsx
// Next.js: ページ全体がクライアントコンポーネント
'use client'
export default function Page() {
  return (
    <div>
      <StaticHeader />
      <InteractiveSearch />
      <StaticContent />
    </div>
  )
}
```

```astro
<!-- Astro: 必要な部分のみハイドレート -->
<Layout>
  <StaticHeader />
  <InteractiveSearch client:load />
  <StaticContent />
</Layout>
```

---

## 5. まとめ & 次章へのブリッジ

Islands Architectureは「Progressive Enhancement」の現代版実装です。静的HTMLをベースラインとし、必要に応じてJavaScriptで機能を拡張する設計思想が、Astroの技術的優位性の根幹となっています。

**次章では**、この理論的基盤の上に構築された「Multi-Framework Magic」を解析します。React、Vue、Svelte、Solidが同一ページで協調動作する仕組み、そして各フレームワークのレンダラー実装の詳細に踏み込みます。