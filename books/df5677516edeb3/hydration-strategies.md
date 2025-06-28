# Hydration Strategies: client:*ディレクティブの内部実装

## 概要

Astroの最も独特な機能の一つが、細かく制御できるHydration戦略です。`client:load`、`client:idle`、`client:visible`、`client:media`といったディレクティブがどのように内部で実装されているかを深掘りします。

## Hydrationディレクティブの種類

### client:load
- **動作**: ページロード直後にHydrate
- **実装**: `window.addEventListener('load', ...)`
- **使用場面**: 即座にインタラクションが必要なコンポーネント

### client:idle
- **動作**: ブラウザがアイドル状態になったときにHydrate
- **実装**: `requestIdleCallback` API使用
- **使用場面**: 重要度の低いインタラクティブ要素

### client:visible
- **動作**: コンポーネントが画面に表示されたときにHydrate
- **実装**: Intersection Observer API
- **使用場面**: スクロールで表示される要素

### client:media
- **動作**: メディアクエリ条件を満たしたときにHydrate
- **実装**: `matchMedia` API
- **使用場面**: レスポンシブ対応のコンポーネント

## 内部実装の詳細

```javascript
// Astroランタイムの簡略化された実装例
class AstroIsland {
  constructor(element, Component, props, hydrate) {
    this.element = element;
    this.Component = Component;
    this.props = props;
    this.hydrate = hydrate;
    this.setupHydration();
  }

  setupHydration() {
    switch(this.hydrate.strategy) {
      case 'load':
        if (document.readyState === 'complete') {
          this.hydrateComponent();
        } else {
          window.addEventListener('load', () => this.hydrateComponent());
        }
        break;
      
      case 'idle':
        if ('requestIdleCallback' in window) {
          requestIdleCallback(() => this.hydrateComponent());
        } else {
          setTimeout(() => this.hydrateComponent(), 200);
        }
        break;
      
      case 'visible':
        const observer = new IntersectionObserver((entries) => {
          entries.forEach(entry => {
            if (entry.isIntersecting) {
              this.hydrateComponent();
              observer.disconnect();
            }
          });
        });
        observer.observe(this.element);
        break;
      
      case 'media':
        const mediaQuery = window.matchMedia(this.hydrate.value);
        if (mediaQuery.matches) {
          this.hydrateComponent();
        } else {
          mediaQuery.addListener((e) => {
            if (e.matches) this.hydrateComponent();
          });
        }
        break;
    }
  }

  hydrateComponent() {
    // フレームワーク固有のHydration処理
    // React: ReactDOM.hydrate
    // Vue: createSSRApp().mount()
    // Svelte: new Component({ target, hydrate: true })
  }
}
```

## パフォーマンスへの影響

### バンドルサイズの最適化
- 必要なコンポーネントのみをバンドル
- Code Splittingによる遅延読み込み
- フレームワーク固有のランタイムも分割

### レンダリングパフォーマンス
- 初期HTMLは即座に表示
- JavaScriptの実行を戦略的に遅延
- ユーザー体験の向上

## 他フレームワークとの比較

### Next.js
```javascript
// Next.jsの場合、全体がHydrate対象
import dynamic from 'next/dynamic'
const DynamicComponent = dynamic(() => import('../components/hello'))
```

### Astro
```astro
<!-- 必要な部分のみを選択的にHydrate -->
<StaticHeader />
<InteractiveComponent client:visible />
<StaticFooter />
```

## 実践的な使い分け

```astro
---
import Header from '../components/Header.astro'
import NavigationMenu from '../components/NavigationMenu.jsx'
import ArticleContent from '../components/ArticleContent.astro'
import CommentSection from '../components/CommentSection.vue'
import RelatedArticles from '../components/RelatedArticles.svelte'
import Newsletter from '../components/Newsletter.jsx'
---

<!-- 静的コンテンツ -->
<Header />

<!-- ナビゲーション：即座に必要 -->
<NavigationMenu client:load />

<!-- 記事本文：静的 -->
<ArticleContent />

<!-- コメント：スクロールして表示されたら -->
<CommentSection client:visible />

<!-- 関連記事：ブラウザがアイドルになったら -->
<RelatedArticles client:idle />

<!-- ニュースレター：モバイルでは非表示 -->
<Newsletter client:media="(min-width: 768px)" />
```

## まとめ

Astroのhydration戦略は、従来のSPAとSSGの中間領域を埋める革新的なアプローチです。各ディレクティブの内部実装を理解することで、より効果的なパフォーマンス最適化が可能になります。