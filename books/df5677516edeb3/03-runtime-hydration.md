---
title: "ランタイムの仕組み：ハイドレーションと対話性の復元"
---

前のセクションでは、Astroがビルド時にいかにして最適化された静的ファイルを生成するかを学びました。しかし、Webサイトはただ表示されるだけではありません。ユーザーのアクションに応答するためのインタラクティブ性が必要です。

この章では、ブラウザが静的なHTMLを受け取った後、どのようにしてJavaScriptを実行し、コンポーネントに"命を吹き込む"のか、その核心である"ハイドレーション"の仕組みを深掘りします。

## 1. 【理論】ランタイム処理と選択的ハイドレーション

ブラウザでページが読み込まれると、まず静的HTMLが即座に表示されます。この段階ではJavaScriptはほとんど実行されないため、ページの表示は極めて高速です。静的コンテンツの表示後、Astroの軽量なランタイムスクリプトが実行され、ページ内に埋め込まれた`<astro-island>`要素をすべて検出し、どのコンポーネントをいつハイドレーションするかの計画を立てます。

Astroが提供する各ハイドレーション戦略は、異なるユースケースに最適化されており、ランタイムスクリプトは各アイランドに指定された戦略に従ってハイドレーションを実行します。例えば、`client:load`戦略はページの主要機能を担うコンポーネントに、`client:visible`戦略は長いページでスクロールされて初めて表示されるフッター部分のコンポーネントなどに適用されます。後者の場合、ブラウザの`Intersection Observer` APIを使用して、ユーザーの操作を妨げることなく、効率的にリソースを読み込みます。

ハイドレーション条件が満たされると、ランタイムは該当するコンポーネントのJavaScriptファイルと、それに対応するフレームワークのレンダラーを動的に読み込みます。その後、各フレームワーク固有のハイドレーション関数（例えばReactの`hydrateRoot`）が呼び出され、サーバーで生成された静的HTMLのDOM構造とクライアントで実行されるコンポーネントの仮想DOMを照合し、イベントリスナーなどをアタッチしてインタラクティブ性を付与します。

## 2. 【実践】最小限のハイドレーションを実装する

理論を学んだところで、これまでの章で作成した仕組みを連携させ、実際にブラウザ上で"島"がインタラクティブになるまでの流れを再現してみましょう。

### ステップ1: ビルドプロセスによるスクリプトの埋め込み

まず、ビルドプロセス（`build.js`）を修正し、ハイドレーションを実行するためのクライアント側スクリプトを、生成するHTMLに埋め込むようにします。また、どのコンポーネントをハイドレーションするかの情報も、`<script type="application/json">`タグを使ってHTML内に埋め込みます。

```javascript
// build.js の build 関数を修正

// ... (前略)
for (const filePath of astroFiles) {
  // ... (中略)
  const finalHtml = compile(fileContent) // コンパイル処理

  // islandを見つけて情報を収集する処理（実際にはコンパイラが担う）
  const islands = findIslandsInHtml(finalHtml)

  // ハイドレーションスクリプトと情報をHTMLに追加
  const htmlWithHydration = addHydrationScript(finalHtml, islands)

  // ... (distへの書き出し処理)
  await fs.writeFile(distPath, htmlWithHydration)
}
// ... (後略)

// HTMLにスクリプトを埋め込むヘルパー関数
function addHydrationScript(html, islands) {
  const islandData = JSON.stringify(islands)
  return `
    ${html}
    <script type="application/json" id="astro-islands">${islandData}</script>
    <script type="module" src="/hydrate.js"></script>
  `
}
```

### ステップ2: クライアント側ハイドレーションスクリプト

次に、クライアント側で動作する `hydrate.js` を作成します。このスクリプトは、`02-islands-deep-dive.md` で作成したものをベースに、HTMLに埋め込まれた情報（`astro-islands`）を読み取って動作するようにしたものです。

```javascript
// public/hydrate.js (distディレクトリにコピーされる想定)

console.log('Hydration script loaded')

// 埋め込まれたアイランド情報を取得・パース
const islandDataElement = document.getElementById('astro-islands')
const islandsToHydrate = JSON.parse(islandDataElement.textContent)

async function hydrateComponent(island) {
  // コンポーネントのJSファイルを動的にインポート
  // 実際のAstroでは、ビルド時にコンポーネント名とファイルパスのマッピングが生成される
  const component = await import(`/components/${island.componentName}.js`)
  
  if (component.default && !customElements.get(island.componentName)) {
    customElements.define(island.componentName, component.default)
    console.log(`${island.componentName} hydrated!`)
  }
}

function initHydration() {
  islandsToHydrate.forEach(island => {
    const element = document.querySelector(island.selector)
    if (!element) return

    if (island.strategy === 'load') {
      hydrateComponent(island)
    }
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
  })
}

// DOMの準備ができてからハイドレーションを開始
document.addEventListener('DOMContentLoaded', initHydration)
```

### ステップ3: 動作確認

ここまでの実装を終え、ビルドを実行して`dist/index.html`をブラウザで開くと、次のような流れが確認できます。

1.  HTMLがレンダリングされ、静的なコンテンツが表示される。
2.  `hydrate.js`が読み込まれ、実行される。
3.  `hydrate.js`が`id="astro-islands"`の`script`タグからどの島をハイドレーションするかの情報を読み取る。
4.  `client:load`が指定されたコンポーネント（例： `<my-header>`）が即座にハイドレーションされる。
5.  ページをスクロールして`client:visible`が指定されたコンポーネント（例： `<my-footer>`）が表示領域に入ると、`IntersectionObserver`がそれを検知し、そのコンポーネントがハイドレーションされる。

これにより、ビルド時に静的生成されたページが、ブラウザ上で動的にインタラクティブ性を取り戻す、というAstroの基本的なランタイムの仕組みを再現できました。

## 3. 【まとめと次章へ】

この章では、Astroのランタイムがどのようにして静的なページにインタラクティブ性をもたらすかを解説しました。必要なコンポーネントだけを、必要なタイミングでハイドレーションする"選択的ハイドレーション"こそが、Astroの優れたパフォーマンスと優れた開発体験を両立させる鍵となっています。次章では、Astroの心臓部であるコンパイラの内部構造に、さらに深く踏み込んでいきます。
