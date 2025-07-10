---
title: "Islands Architectureの理論と実装"
---

第1章ではJavaScript肥大化という課題を位置付け、その解決策の方向性として「送るコードを減らす」思想を確認しました。本章では、その思想を具体的な形にしたIslands Architectureを、設計の意図からビルドやランタイムの内部処理まで通しで解説します。

## この章の目的
1. アイランドアーキテクチャが採用された理由とメリットを理解する。
2. 理論と実測値の両面から読み解く。
3. ハイドレーション戦略がブラウザでどのようにトリガされ、ユーザー体験を最適化するか。
4. 手法と比較し、共通する最適化原則と設計トレードオフを整理する。

## Islands Architectureとは

Astroのパフォーマンスを支える最も重要なコンセプトが「Islands Architecture（アイランドアくファリルはキテクチャ）」です。これは、Webページを静的なHTMLの「海」と、その中に浮かぶインタラクティブな「島（アイランド）」として構成する考え方です。

- **海（Sea）** は、ページの大半を占める、静的でインタラクティブ性を持たないHTML。サーバーでレンダリングされ、ブラウザに配信されます。
- **島（Island）** は、ボタン、画像カルーセル、検索フォームなど、インタラクティブなUIコンポーネント。これらの「島」だけが、クライアントサイドでJavaScript（JS）を読み込み、ハイドレーション（静的なHTMLにイベントリスナーなどを付与して動的にすること）されます。

このアーキテクチャにより、Astroはデフォルトで不要なJavaScriptを一切クライアントに送信しない「Zero-JS」を実現し、高速なページ読み込みを可能にしています。

## 最小限のIslands Architectureを実装する

それでは、このIslands Architectureの仕組みを、簡単なコードで再実装してみましょう。目標は、特定のカスタム要素（例えば `<my-counter>` のような）を「島」として認識し、その島に対応するJavaScriptだけをクライアントで読み込む仕組みを作ることです。

### 1. サーバーサイドでHTMLをスキャンする

まず、ビルド時やリクエスト時に、サーバーサイドで生成されたHTMLから「島」を見つけ出す必要があります。ここでは、正規表現を使って特定のカスタム要素を探索する簡単なスキャナを考えます。

```js:server-scanner.js
// 探索対象のHTML（Astroコンポーネントから生成されたと仮定）
const html = `
  <html>
    <body>
      <h1>静的なコンテンツ</h1>
      <p>ここはインタラクティブではありません。</p>
      
      <!-- これがインタラクティブな島 -->
      <my-counter client:load></my-counter>
      
      <p>ここも静的なコンテンツです。</p>
      
      <!-- もう一つの島 -->
      <my-counter client:visible></my-counter>
    </body>
  </html>
`

// client:* ディレクティブを持つカスタム要素を見つける正規表現
const islandRegex = /<([a-zA-Z0-9-]+)(\s+[^>]*?client:(load|visible|idle|media)=?["\\]?["\\]?[^"\\]*?["\\]?["\\]?[^>]*?)>.*?<\/\1>/gs

const islands = []
let match
while ((match = islandRegex.exec(html)) !== null) {
  islands.push({
    componentName: match[1],
    html: match[0],
    directive: match[2], // client:load など
  })
}

console.log('見つかった島:', islands)
```

このコードは、`client:*`という特別なディレクティブを持つカスタム要素をHTMLから探し出します。Astroでは、このディレクティブが、コンポーネントをインタラクティブな島として扱うための目印となります。

### 2. クライアントサイドでハイドレーションする

次に、サーバーサイドで見つけた島を、クライアントサイドで動的にするためのスクリプトを考えます。このスクリプトは、ページの読み込みが完了した際に、対応するコンポーネントのJavaScriptをインポートし、カスタム要素を「ハイドレーション」します。

```js:client-hydrator.js
// サーバーから渡された情報（どのコンポーネントをいつハイドレーションするか）
const islandsToHydrate = [
  { componentName: 'my-counter', selector: 'my-counter[client\\:load]', strategy: 'load' },
  { componentName: 'my-counter', selector: 'my-counter[client\\:visible]', strategy: 'visible' },
]

// コンポーネントのコードを動的にインポートする関数
async function hydrateComponent(island) {
  // 対応するコンポーネントのJSファイルを動的にインポート
  // 実際には、ビルド時にコンポーネント名とファイルパスのマッピングが生成される
  const component = await import(`./components/${island.componentName}.js`)
  
  // カスタム要素を登録
  if (component.default && !customElements.get(island.componentName)) {
    customElements.define(island.componentName, component.default)
  }
  console.log(`${island.componentName} がハイドレーションされました。`)
}

// ハイドレーション戦略に基づいて実行
function initHydration() {
  islandsToHydrate.forEach(island => {
    if (island.strategy === 'load') {
      // ページ読み込み完了時に実行
      window.addEventListener('DOMContentLoaded', () => hydrateComponent(island))
    }
    if (island.strategy === 'visible') {
      // 要素がビューポートに入ったら実行
      const elements = document.querySelectorAll(island.selector)
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            hydrateComponent(island)
            observer.unobserve(entry.target)
          }
        })
      })
      elements.forEach(el => observer.observe(el))
    }
  })
}

initHydration()
```

### 3. コンポーネントの実装

最後に、インタラクティブな島となるカウンターコンポーネント自体を実装します。これは、標準的なWeb Components（カスタム要素）として作成します。

```js:components/my-counter.js

class MyCounter extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
    this.count = 0
  }

  connectedCallback() {
    this.render()
    this.shadowRoot.querySelector('button').addEventListener('click', () => {
      this.count++
      this.render()
    })
  }

  render() {
    this.shadowRoot.innerHTML = `
      <style>
        div { border: 1px solid black; padding: 1rem; }
      </style>
      <div>
        <p>Count: ${this.count}</p>
        <button>Increment</button>
      </div>
    `
  }
}

export default MyCounter
```

Islands Architectureが提起した「海と島」という比喩は、2025年現在、主要フレームワークの設計思想そのものを塗り替えつつあります。ページ全体を一気にハイドレートする従来型の手法では、ダウンロードと実行の両面で過剰なJavaScriptを抱え込みやすく、初回表示と操作可能になるまでの間に大きなギャップが残ります。ここでは、React、Vue、Svelteが近年どのようにして「必要な場所だけに実行時ロジックを届ける」方向へ舵を切ったかを、順に掘り下げます。

### React

React 18までは`hydrateRoot`を境にページ全体をまとめてハイドレートし、`React.lazy`や`import()`で初期バンドルを小さくしても「結局どこかで大量のJSを実行する」問題が残っていました。2025年6月に安定版となったReact 19では、その前提そのものが反転します。Server Componentsと`"use client"`ディレクティブが安定化し、クライアントに届くのはインタラクティブな境界部分だけ、その他はHTMLと最小限のシリアライズ済みデータに置き換わります。Next.js 15のApp Routerでは、この境界を並列ストリーミングで送り出すため、HTMLが即座に描画され、必要分のJSが後から到着する流れが標準化されました。フォーム送信やペンディング表示を`useTransition`に委ねることで、クライアント側fetchも一段と減らせるようになり、実測で送信JSが半分前後まで圧縮された事例が報告されています。

### Vue

Vue 3自体はランタイムが小さいものの、アプリ全体を一括ハイドレートすればReactと同じ課題を抱えます。これに対しNuxt 3以降は`*.server.vue`コンポーネントと`<NuxtIsland>`を組み込み、島の中身を静的HTMLとして返し、クライアントJSをまったく送らない設計を正式機能にしました。動的箇所が必要になった瞬間にだけ再フェッチして差し替えるため、同じSSRでも再ハイドレートのJS量が劇的に小さくなります。さらに`vue-lazy-hydration`の`<LazyHydrate>`ラッパーを併用すれば、ビューポートに入る直前のタイミングで初めてハイドレートを走らせることも可能です。2025年7月にRC入りしたNuxt 4では、これらの機能がデフォルト設定のまま組み合わさり、サーバーコンポーネントと遅延ハイドレーションの相乗効果で、送信JSを一桁キロバイトまで押し下げたケースも報告されています。

### Svelte 

Svelteは「コンパイル時にランタイムを捨てる」という思想により、初期からバンドルサイズが小さいのが強みでした。ただしSvelteKitでSSR後にページ全体を即ハイドレートする限り、「必要ないのに実行されるJS」をゼロにはできません。次期メジャーであるSvelte 5はRunes導入で依存解析が細粒度化し、コンパイラ側で「この要素にはJS不要」と判断した箇所を自動的に静的化する準備が進んでいます。GitHub上では`partial-hydration-sk`など実験的プラグインが既に動いており、`hydrate: false`をデフォルトにして`<Island>`で囲んだ部分だけクライアントJSを注入するアプローチが実用域に入りつつあります。Runesを乱用すると再びバンドルが膨らむため、`$state`よりも`$derived`や`$effect`で依存を局所化し、コンパイラに「不要コードを捨てさせる」書き方が推奨され始めています。

### 共通する最適化の流れ

三つのエコシステムは方向性こそ違えど、「HTMLとJSの境界を宣言的に示す」「ビューポートやユーザー操作をトリガーに初めてハイドレートする」という二つの原則で収束しています。その結果、送信するJavaScriptの総量は、かつてのSPA全体ハイドレート方式と比べて桁違いに削減され、同じSSR/SSGでもTTIが短縮されるだけでなく、長時間の実行時メモリも抑制できます。今後はRustベースのRolldownのような並列ビルド対応バンドラが、未使用エクスポートの除去をより確実にし、さらにJS量を削る役割を担う見込みです。

### まとめ

ReactはServer Componentsの安定化で「デフォルトはサーバー」に転換し、VueはNuxt IslandとLazy Hydrationを公式機能に取り込み、SvelteはRunesによるコンパイラ強化で自動部分ハイドレーションに近づいています。どのコミュニティも、静的HTMLを守りながら「いつ、どこへ、どれだけJSを流すか」を宣言的に制御できる方向へと収束しており、Islands Architectureが描いた「海と島」の境界は、単なる比喩を超えてWebフレームワークの共通設計原則になりつつあります。


何ができるようになってほしいのか
- AstroのIslands Architectureの基本概念を理解。
- ビルドパイプラインの内部実装を把握できる。
- 