---
title: "ハイドレーション戦略とパフォーマンス"
---

アイランドアーキテクチャのパフォーマンスを最大化する鍵は、JavaScriptの実行タイミングをいかに最適化するかにかかっています。この章では、Astroが提供する多彩な"ハイドレーション戦略"を1つずつ詳しく見ていきます。それぞれの戦略がどのようなユースケースのために設計され、内部的にどのブラウザAPIを利用しているのかを理解することで、読者は自身のサイトに最適なパフォーマンスチューニングを施す知識を身につけることができます。

## 1. 【課題】ハイドレーションのコスト

たとえアイランドアーキテクチャでJavaScriptの量を削減したとしても、その実行（ハイドレーション）がページの初期ロード時に集中すれば、メインスレッドがブロックされ、ユーザーの操作に対する応答性が損なわれる可能性があります。例えば、ページの全インタラクティブ要素を`client:load`で読み込んでしまうと、それは従来のSPAが抱えていた問題と何ら変わりません。LighthouseのレポートでTotal Blocking Time (TBT)やInteraction to Next Paint (INP)といった指標が悪化する主な原因は、この初期ロード時のメインスreadの混雑にあります。

## 2. 【解決策】選択的ハイドレーションの戦略

Astroは、この課題を解決するために、開発者がコンポーネントごとにハイドレーションのタイミングを宣言的に指定できる、複数の`client:*`ディレクティブを提供します。これにより、本当に必要な時までJavaScriptの実行を遅延させることが可能になります。

ページのヘッダーにあるナビゲーションメニューのように、すぐに操作可能であるべきUIには`client:load`を選択するのが適切です。これは、ページの初期ロードが完了した直後にハイドレーションを実行します。

一方で、記事ページの下部にある関連記事一覧やコメント欄のような、すぐに表示される必要のないコンポーネントには`client:idle`が有効です。この戦略は、ブラウザのメインスレッドが手隙になったタイミングでハイドレーションを行う`requestIdleCallback` APIを利用するため、ページの初期表示やユーザーの操作を妨げることがありません。

さらに、画像ギャラリーや動画プレーヤーのように、それ自体がページの大部分を占めるような重いコンポーネントには`client:visible`が最適です。これはブラウザの`IntersectionObserver` APIを用いて、コンポーネントがユーザーのビューポートに入るまで、一切の処理を遅延させます。これにより、ユーザーがスクロールしてその部分に到達するまで、関連するJavaScriptはダウンロードすらされないため、データ通信量の節約にも繋がります。

最後に、`client:media`というユニークな戦略も存在します。これは特定のCSSメディアクエリが一致した場合にのみハイドレーションを実行するもので、例えば"PCの画面幅でのみ表示されるサイドバーメニュー"といったレスポンシブデザインに特化した最適化を実現します。内部的には`matchMedia` APIがこの挙動を支えています。

これらの戦略を適切に使い分けることで、開発者はパフォーマンスへの影響を最小限に抑えながら、リッチなインタラクティブ性を提供できるのです。

## 3. 【実践】高度なハイドレーション戦略を実装する

理論を学んだところで、これまでの章で作成したハイドレーションスクリプトを拡張し、`client:idle`と`client:media`戦略をサポートしてみましょう。

### `hydrate.js`の拡張

`03-runtime-hydration.md`で作成した`hydrate.js`に、新たな戦略を処理するロジックを追加します。`client:idle`には`requestIdleCallback`を、`client:media`には`window.matchMedia`を使用します。

```javascript
// public/hydrate.js (拡張版)

console.log('Hydration script loaded')

const islandDataElement = document.getElementById('astro-islands')
const islandsToHydrate = JSON.parse(islandDataElement.textContent)

async function hydrateComponent(island) {
  // (実装は変更なし)
  const component = await import(`/components/${island.componentName}.js`)
  if (component.default && !customElements.get(island.componentName)) {
    customElements.define(island.componentName, component.default)
    console.log(`${island.componentName} hydrated using strategy: ${island.strategy}`)
  }
}

function initHydration() {
  islandsToHydrate.forEach(island => {
    const element = document.querySelector(island.selector)
    if (!element) return

    // client:load 戦略 (変更なし)
    if (island.strategy === 'load') {
      hydrateComponent(island)
    }

    // client:visible 戦略 (変更なし)
    if (island.strategy === 'visible') {
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            hydrateComponent(island)
            observer.unobserve(entry.target)
          }
        })
      })
      observer.observe(element)
    }

    // 【追加】client:idle 戦略
    if (island.strategy === 'idle') {
      if ('requestIdleCallback' in window) {
        window.requestIdleCallback(() => hydrateComponent(island))
      } else {
        // フォールバックとして、200ms遅延させて実行
        setTimeout(() => hydrateComponent(island), 200)
      }
    }

    // 【追加】client:media 戦略
    if (island.strategy === 'media') {
      // island.value にメディアクエリ文字列が含まれる想定 (例: '(max-width: 768px)')
      const mediaQuery = window.matchMedia(island.value)
      if (mediaQuery.matches) {
        hydrateComponent(island)
      } else {
        // メディアクエリの状態変化を監視
        mediaQuery.addEventListener('change', (e) => {
          if (e.matches) {
            hydrateComponent(island)
          }
        }, { once: true }) // 一度実行されたらリスナーを解除
      }
    }
  })
}

document.addEventListener('DOMContentLoaded', initHydration)
```

この修正により、ハイドレーションスクリプトは4つの異なる戦略を扱えるようになりました。ビルドプロセス側で、`client:media`が指定されたアイランドの情報にメディアクエリ文字列（例: `(max-width: 768px)`）を`value`プロパティとして含めるようにすれば、このスクリプトはレスポンシブなハイドレーションを実現できます。

## 4. 【実践演習】Lighthouseで効果を測定する

これらの戦略の効果を実感するために、簡単な計測演習を行いましょう。まず、インタラクティブなコンポーネントを複数配置したテストページを作成し、すべてのコンポーネントに`client:load`を指定してビルドします。次に、Google Chromeの開発者ツールに統合されているLighthouseを実行し、パフォーマンススコア、特にTBTの数値を記録します。その後、各コンポーネントの役割に応じてディレクティブを`client:idle`や`client:visible`に書き換え、再度Lighthouseで計測します。2つの結果を比較すれば、ハイドレーション戦略の最適化が、具体的なパフォーマンス指標の改善にどれほど貢献するかが明確にわかるはずです。

## 5. 【まとめと次章へ】

この章では、Astroが提供する多彩なハイドレーション戦略と、それらがパフォーマンスに与える影響について学びました。JavaScriptの実行タイミングをコンポーネント単位で細かく制御する能力こそが、Astroの高速性を支える重要な柱の1つです。

次章では、Astroのアーキテクチャのもう1つの心臓部である"Viteプラグインアーキテクチャ"に焦点を当てます。AstroがどのようにしてViteの強力なエコシステムを拡張し、独自のビルドプロセスを実現しているのか、その内部構造を解き明かしていきます。