---
title: "ビルドの仕組み：静的HTML生成"
---

アイランドアーキテクチャを実現するための最初のステップは、ブラウザに渡すファイルを事前に準備する"ビルドプロセス"です。この章では、`.astro`ファイルや各種フレームワークのコンポーネントが、どのようにして最適化された静的なHTML、CSS、そして最小限のJavaScriptに変換されるのか、その詳細なステップを解き明かします。このビルドプロセスを理解することは、Astroがなぜ"デフォルトでZero-JS"を実現できるのかを知るための鍵となります。
AstroがZero-JSを成立させるためにビルド段階でどこまで責任を負っているか"を理解させることです。単にソースをHTMLに変換する手順の列挙ではなく、ビルドが果たす役割と、その結果として得られるパフォーマンス上の利得を結びつける

## この章の目的
1. ビルドパイプライン（ファイル検出→レンダラー選定→SSR→Viteバンドル）が担う処理の流れを具体的に追う。
2. コンポーネントを静的HTMLと <astro-island> プレースホルダーへ変換する過程で、どこまでJSをそぎ落とせるかを理解する。
3. ツリーシェイク・コードスプリッティング・アセット最適化が最終バンドルとビルド時間に及ぼす効果を数値で把握する。
4. 他フレームワークのビルドフェーズと比較し、"事前に削る"思想の違いを整理する。

## 1. 【理論】静的生成パイプラインの内部

Astroのビルドプロセスは、プロジェクト内のソースファイルを包括的に解析することから始まります。この解析フェーズでは、Astroファイルだけでなく、React、Vue、Svelteなど、`astro.config.mjs`に登録されたインテグレーションを通じてサポートされるすべてのフレームワークのコンポーネントファイルが検出対象となります。

ファイルが検出されると、次に各コンポーネントに対応する"レンダラー"が特定されます。例えば、ファイルがReactのJSX構文を含んでいればReactレンダラーが選ばれ、その`renderToStaticMarkup`のようなサーバーサイドレンダリング用の関数が呼び出されます。この関数は各フレームワークの標準的な機能（例えばReactでは`ReactDOMServer.renderToString()`）を利用しており、コンポーネントを純粋なHTML文字列に変換します。この段階ではクライアントサイド専用のフック、例えばReactの`useEffect`などは実行されず、安全に初期HTMLが生成されることが保証されています。

インタラクティブな"島"としてマークされたコンポーネントに対しては、ビルドプロセスは特別な処理を行います。後のハイドレーションで必要となる情報をすべて属性として持つ、`<astro-island>`というカスタム要素でそのコンポーネントの初期HTMLをラップします。この時、コンポーネントに渡されるpropsは、単純な`JSON.stringify`では扱えないDateオブジェクトやRegExpなども処理できる、Astro独自の直列化システムによって安全に文字列化されます。

最終的に、Astroは内部でViteを呼び出し、これらの処理を経て生成されたHTMLや、島のためのJavaScriptチャンクをバンドルします。Viteの強力なTree Shaking機能により、島で使われていないReactのフック（例えば`useContext`）などは最終的なバンドルから除去され、各島のJavaScriptサイズは最小限に抑えられます。この一連の流れが、Astroの高速なサイト生成の根幹を成しているのです。

## 2. 【実践】最小限のビルドプロセスを実装する

理論を学んだところで、次はこの知識を元に、非常にシンプルなビルドプロセスをNode.jsで実装してみましょう。ここでの目標は、特定のディレクトリ（例： `src`）にある`.astro`ファイルをすべて探し出し、前の章で作成したコンパイラを使ってHTMLに変換し、結果を別のディレクトリ（例： `dist`）に出力することです。

### 準備: ファイル構造

まず、次のようなファイル構造を準備します。

```
/my-astro-project
|-- /src
|   |-- index.astro
|   `-- about.astro
|-- /dist
|-- build.js
`-- mini-astro-compiler.js (前の章で作成)
```

### ステップ1: ファイルの探索

`build.js`の中で、`src`ディレクトリ内の`.astro`ファイルを再帰的に探し出す処理を実装します。Node.jsの標準モジュールである`fs`と`path`を使います。

```javascript
// build.js

import fs from 'fs/promises'
import path from 'path'
import { compile } from './mini-astro-compiler.js' // 前の章のコンパイラをインポート

const SRC_DIR = 'src'
const DIST_DIR = 'dist'

async function findAstroFiles(dir) {
  const entries = await fs.readdir(dir, { withFileTypes: true })
  const files = await Promise.all(entries.map(entry => {
    const fullPath = path.join(dir, entry.name)
    if (entry.isDirectory()) {
      return findAstroFiles(fullPath)
    }
    if (entry.isFile() && entry.name.endsWith('.astro')) {
      return fullPath
    }
    return []
  }))
  return files.flat()
}
```

### ステップ2: コンパイルと出力

見つけた`.astro`ファイルそれぞれに対して、コンパイラを実行し、結果を`dist`ディレクトリに書き出す処理を追加します。

```javascript
// build.js (続き)

async function build() {
  try {
    // distディレクトリがなければ作成
    await fs.mkdir(DIST_DIR, { recursive: true })

    const astroFiles = await findAstroFiles(SRC_DIR)
    console.log(`Found ${astroFiles.length} .astro files.`)

    for (const filePath of astroFiles) {
      console.log(`Building: ${filePath}`)
      const fileContent = await fs.readFile(filePath, 'utf-8')
      
      // コンパイラでHTMLを生成
      const finalHtml = compile(fileContent)
      
      // 出力先のパスを計算
      const relativePath = path.relative(SRC_DIR, filePath)
      const distPath = path.join(DIST_DIR, relativePath.replace(/\.astro$/, '.html'))
      
      // 出力先のディレクトリを作成
      await fs.mkdir(path.dirname(distPath), { recursive: true })
      
      // HTMLファイルとして書き出し
      await fs.writeFile(distPath, finalHtml)
      console.log(`  -> Output: ${distPath}`)
    }

    console.log('\nBuild complete!')
  } catch (error) {
    console.error('\nBuild failed:', error)
  }
}

build()
```

この`build.js`を`node build.js`コマンドで実行すると、`src`ディレクトリの`.astro`ファイルがコンパイルされ、`dist`ディレクトリに同名の`.html`ファイルとして出力されます。これは、`astro build`コマンドが行っていることの、最も基本的な部分を抜き出したものと言えます。

## 3. 【まとめと次章へ】

この章では、Astroがソースコードを解析し、最適化された静的ファイルを生成するまでのビルドプロセスの流れを解説しました。サーバーサイドで可能な限りの処理を完了させることが、Astroの高速化の秘訣です。

次章では、こうして生成されたファイルがブラウザに読み込まれた後、どのようにしてインタラクティブ性を取り戻すのか、"実行時の仕組みとハイドレーション"について詳しく見ていきます。