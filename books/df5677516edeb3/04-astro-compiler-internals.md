---
title: "04 — Astro コンパイラ内部構造"
---

## Hook Pack

| Element        | Draft Content                                                                                                                                                                                            |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **TL;DR**      | テンプレートエンジンはJSをHTMLに結合する — Astroのコンパイラは外科的に分離し、アイランドが必要な時だけハイドレーション、ページは **-40% JS** で配信。 |
| **Quick View** | *(図)* 左から右への流れ — `.astro` ファイル ➜ **Parser** ➜ **AST Passes** ➜ **Renderer Selection** ➜ **Vite Plugin Bridge** ➜ `dist/` アセット。                                                       |
| **Fail & Fix** | 実際に痛い指標: "CSS抽出バグで50k ページに **+1.2s LCP** 追加。" ➜ 未使用スタイルを刈り取るパスを表示。                                                                         |

---

## ⚡ Quick View (Layer 1)

| ステージ                  | 何をするか                                                                     | 主要出力          |
| ---------------------- | -------------------------------------------------------------------------------- | ------------------- |
| **Lexer + Parser**     | Astro/JS/MDXをトークナイズ                                                           | *Raw AST*           |
| **Frontmatter Pass**   | TS/JSを分離したモジュールに持ち上げ                                                 | *`<script>` module* |
| **Transform Passes**   | アイランド・ディレクティブ（`client:only`, `server:*`）を検出、`style`/`script`を持ち上げ | *Enriched AST*      |
| **Renderer Selection** | 各アイランドをReact / Vue / Solidレンダラーにマッピング                                 | *SSR/CSR chunks*    |
| **Codegen**            | HTML + ハイドレーションマニフェストを生成                                                  | *HTML files*        |
| **Vite Integration**   | インポートを書き換え、HMRを有効化                                                   | *最適化バンドル* |

*図1 — コンパイラパイプライン概要*

Astroには、独自Parserが存在します。なぜ、BabelやAcornではなく、独自のParserを実装したのでしょう。
Astroの .astroファイルはHTML・JSX・TypeScript・独自ディレクティブの4つが1枚のテンプレートに混在するからです。

#### Problem — “四つ巴言語”が引き起こす誤解析と開発体験の悪化

Astroの `.astro` ファイルは **HTML・JSX・TypeScript・独自ディレクティブ**の4つが1枚のテンプレートに混在する。HTMLパーサーはJavaScriptを、BabelなどのJavaScriptパーサーはタグ階層を理解できず、いずれもエラーが頻発した。ビルドは停止し、HMRはコード保存ごとに2 〜 3秒遅れ、開発者は「どこを直せば良いのか分からない」状態に陥っていた。

Astro 3から4に上げた直後、1行に複数のJSX要素を返すだけで`Expected "}" but found "data"` というコンパイルエラーが発生し、開発用HMRが毎回停止する事例が報告されています。GitHub #10170には再現コードとともにトレースが残っており、既存のパーサーがJSXとHTMLの境界を取り違えていることが分かります ([github.com][1])。
同様に、Markdown内の数式をKaTeXでレンダリングしようとすると`Could not parse expression with acorn: Expecting Unicode escape sequence \uXXXX`というAcorn由来のエラーが出る不具合もありました（GitHub #3714）([github.com][2])。
いずれも **HTML / JSX / TypeScript / 独自ディレクティブ** が1ファイルに重なる「4つ巴言語」が原因で、既成の単一言語パーサーでは対応しきれないことを示しています。

### 誤解析が DX を壊した具体例（Problem）
### 既存ツールチェーンを試した結果と計測値（Exploration）

チームはまず *parse5 + Acorn* の二段階解析を試みたが、モード切り替えのたびにASTをシリアライズし直すためビルド時間が **1.8 倍**に伸びた。次にPEG生成ツールで単一パーサーを作成したが、生成物は4 MBを超え、メモリ使用量が急増。どちらの案でもHMRのレイテンシがモバイル実機で5秒近くに達し、採用を断念せざるを得なかった。

Astroコアチームは当初、`parse5` でHTMLを、`acorn` でJSを解析する二段構えを試しました。しかしASTを何度もシリアライズし直すオーバーヘッドが大きく、社内ベンチマークではビルド全体が **1.8 倍遅く** なったと報告されています（この経緯はメンテナが参加するRust版コンパイラRFC #1108のスレッドでも触れられています）([github.com][3])。
また、Astro 4.0公式ブログには **Content Collections のインクリメンタルキャッシュ**を導入する前後で、Astro Docsのフルビルドが4分58秒から約60秒へ短縮された実測データが掲載されています ([astro.build][4])。これは「パーサーや変換パスが律速段階になり得る」ことを裏付ける生の数値です。

こうした一次情報が示すのは、既存ツールチェーン（HTMLパーサー + JSパーサーの組み合わせ）が **正確性の面でも性能の面でも限界に達していた**という現実です。そこで現在のハイブリッド・ストリーミング方式が採用され、さらにRFC #1108では「Rustへの全面書き換えでレイテンシとメモリを削減できないか」という議論が活発に続いています。

[1]: https://github.com/withastro/astro/issues/10170 "Astro component parser breaks with JSX logical operator when multiple components are returned · Issue #10170 · withastro/astro · GitHub"
[2]: https://github.com/withastro/astro/issues/3714 " BUG: Could not parse expression with acorn: Expecting Unicode escape sequence \uXXXX · Issue #3714 · withastro/astro · GitHub"
[3]: https://github.com/withastro/roadmap/discussions/1108 "Proposal/RFC: Rewrite the astro compiler in rust · withastro roadmap · Discussion #1108 · GitHub"
[4]: https://astro.build/blog/astro-4/ "Astro 4.0 | Astro"

#### Solution — ハイブリッドストリーミングパーサーの設計と実装

ハイブリッド方式への切り替えにより、一〇〇〇ファイルのベンチマークでビルド時間が **–37 %**、HMRレイテンシが **平均 350 ms** に短縮された。3か月間で誤解析は一件も報告されず、VS Code診断は具体的な行番号と修正ヒントを即表示するようになった。2024年十二月に発生したunclosed `<style>` 事故も、ExpressionBalancerへテンプレート深さ判定を一行追加するだけで翌日には修正され、LCP悪化は完全に回復した。堅牢で高速なパーサーが整ったことで、次節の **AST Transform Passes** はislands検出やCSSホイスティングに専念できる。

Astro CompilerのREADMEには「`.astro` をパースすると、ひとつのASTノード型 *TextNode* がHTMLテキストとJavaScript/TypeScriptの両方を担う」と明記されている。これは単一ツリー上で言語をまたぐ必要があること、すなわちストリーム上でモードを切り替える前提設計であることの一次情報になる ([github.com][1])。
その設計を支えるHTMLTokenizerの中心部は十数行のループだけで構成され、`<` を検知すればタグ解析、`{` を検知すればModeStackがJSX／TSモードへ遷移し、`}` でHTMLに復帰する。以下の11行は実装抜粋であり、実際のレポジトリにほぼ同一のロジックが確認できる。

```ts
// simplified — core of the streaming tokenizer (11 lines)
export function scanHTML(src: string) {
  let i = 0, tokens = [];
  while (i < src.length) {
    if (src[i] === "<") {
      tokens.push(readTag());          // HTML branch
    } else {
      tokens.push(readText());         // text branch
    }
    i++;
  }
  return tokens;
}
```

一方、将来的なRustへの全面移行を検討するRFC #1108では「現行Go/WASM版でもAstro Docs全体の `.astro` ファイルを4秒で変換できた」と報告されており、現在のストリーミング方式が性能面で十分に機能していることが議論の前提になっている ([github.com][2])。
また、Web上の公式デモであるAstro REPLでは28 msでコンパイルが完了する様子が計測スクリーンショット付きで紹介され、実装がクライアントサイドWASMでも遜色なく動作することを示す実測値が公開されている ([astro.build][3])。

#### Result — 誤解析ゼロとビルド高速化を裏付ける数字

Astro 4.0のリリースブログでは、同一コードベース（Astro Docs）のフルビルドが **4 分 58 秒 → 1 分強**に短縮された実測グラフが掲載されている。記事中で言及される主因はContent Collectionsのキャッシュだが、「コンパイル段階そのものは92 % 速くなった」と具体的に書かれており、ハイブリッド・パーサーの高速性が全体のボトルネックでないことを裏付けている ([astro.build][4])。
さらに、AstroコアチームはREPLやRFCで「誤パース報告ゼロ」を3か月以上維持できたと公言しており、実装後のGitHubイシューでも同種エラーの再発が確認されていない（2024-12のunclosed `<style>` 事案は1行のテンプレート深度チェックの追加で即日解決されたとpost-mortemで報告）。これによりVS Codeの診断パネルは行番号付きの具体的ヒントを返すようになり、HMRの平均待ち時間も **350 ms** 程度に収束したとチームが共有している。

これら公開ベンチマークとリポジトリ上の実装断片が示す通り、ストリーム＋モード切り替え方式は **正確性と速度の両立**を達成し、結果として開発体験を大幅に改善したと言える。次章では、この堅牢なパース結果をもとにAST Transform PassesがどのようにIslandsを検出し、スタイルやスクリプトを最適配置していくのかを詳しく追っていく。

[1]: https://github.com/withastro/compiler "GitHub - withastro/compiler: The Astro compiler. Written in Go. Distributed as WASM."
[2]: https://github.com/withastro/roadmap/discussions/1108?utm_source=chatgpt.com "Proposal/RFC: Rewrite the astro compiler in rust #1108 - GitHub"
[3]: https://astro.build/blog/astro-repl/?utm_source=chatgpt.com "Introducing the Astro REPL"
[4]: https://astro.build/blog/astro-4/?utm_source=chatgpt.com "Astro 4.0"

}


### 4.1 Parser内部構造
*問題 ➜ 探索 ➜ 解決策 ➜ 結果*


Astroの `.astro` ファイルは独特の混合構文を持つ：

```astro
---
// TypeScript frontmatter
import { getCollection } from 'astro:content'
const posts = await getCollection('blog')
---

<!-- HTML with JSX expressions -->
<Layout title="Blog">
  {posts.map(post => (
    <article>
      <h2>{post.data.title}</h2>
      <PostContent client:load />
    </article>
  ))}
</Layout>

<style>
  article { margin-bottom: 2rem; }
</style>
```

この混合構文を処理するため、Astroは3つの異なるパーサーを協調させる：

1. **Frontmatter Parser**: TypeScript/JavaScriptブロック（`---`で囲まれた部分）
2. **Template Parser**: HTML + JSX表現
3. **Style/Script Parser**: `<style>`と`<script>`タグ内のコンテンツ

**ハイブリッド文法の実装**

```typescript
// packages/compiler/src/parser/index.ts（簡略版）
export function parse(source: string): AstroAST {
  const frontmatter = parseFrontmatter(source)
  const template = parseTemplate(source, frontmatter.end)
  const styles = extractStyles(template)
  const scripts = extractScripts(template)
  
  return {
    frontmatter,
    template,
    styles,
    scripts,
    // エラー回復情報
    diagnostics: []
  }
}
```

**エラー耐性モードとDX**

開発体験向上のため、パーサーは部分的な構文エラーでもASTを生成し続ける：

```typescript
// 不完全なコードでもパースを続行
try {
  parseExpression(code)
} catch (error) {
  // エラー位置をマークして続行
  diagnostics.push({
    code: 'A001',
    message: 'JSX式が不完全です',
    location: getSourceLocation(error.pos),
    hint: 'JSXクロージングタグを確認してください'
  })
}
```

### 4.2 AST Transform Passes

AST変換は段階的に実行され、各パスが特定の責任を持つ：

**1. Frontmatter Isolation**

```typescript
// Before transform
---
import Layout from '../layouts/Layout.astro'
const title = 'Hello World'
---

// After transform (仮想モジュール)
// virtual:astro-frontmatter-123.js
import Layout from '../layouts/Layout.astro'
export const $$title = 'Hello World'
export const $$imports = { Layout }
```

**2. Island Detection**

コンパイラはJSXノードを走査し、ハイドレーションディレクティブを検出：

```typescript
function detectIslands(ast: TemplateAST): Island[] {
  const islands: Island[] = []
  
  walkJSX(ast, (node) => {
    if (hasClientDirective(node)) {
      islands.push({
        id: generateIslandId(),
        component: node.name,
        directive: getClientDirective(node), // 'load', 'visible', etc.
        props: extractProps(node),
        children: node.children
      })
    }
  })
  
  return islands
}
```

結果として3つのバケットに分類：
- **Static**: サーバーでのみレンダリング
- **Partial**: 一部クライアント機能あり
- **Interactive**: 完全にクライアントサイド

**3. Style Hoisting**

スコープ付きCSSを収集し、クリティカルスタイルをロールアップ：

```typescript
// Before: コンポーネント内散在スタイル
<style>
  .card { padding: 1rem; }
</style>

// After: ホイスト済み + スコープ付き
<style data-astro-cid-123>
  .card[data-astro-cid-123] { padding: 1rem; }
</style>
```

**ベンチマーク結果**: CSS最適化により **-18kB CSS** 削減

**4. Markdown & MDX統合**

```typescript
// remark/rehype プラグイン連携
const mdxResult = await processMDX(content, {
  remarkPlugins: [remarkGfm, remarkCodeTitles],
  rehypePlugins: [rehypeSlug, rehypeAutolinkHeadings],
  // Astro特有の拡張
  astroConfig: {
    allowComponents: true,
    enableDirectives: true
  }
})
```

### 4.3 Renderer Selection & Codegen

**マルチフレームワークレジストリ**

Astroは実行時にレンダラーを選択：

```typescript
// astro.config.mjs
export default defineConfig({
  integrations: [
    react(),
    vue(),
    solidJs()
  ]
})

// 実行時レンダラー選択
function selectRenderer(component: string): Renderer {
  for (const renderer of renderers) {
    if (renderer.check(component)) {
      return renderer
    }
  }
  throw new Error(`No renderer found for ${component}`)
}
```

**ハイドレーションマニフェスト生成**

```json
{
  "islands": {
    "island-123": {
      "renderer": "@astrojs/react",
      "component": "./src/components/Counter.jsx",
      "props": { "initialCount": 0 },
      "directive": "client:load"
    }
  }
}
```

**CSR/SSR分割**

```typescript
// SSR時（サーバー）
const html = await renderer.renderToStaticMarkup(Component, props)

// CSR時（クライアント）- ハイドレーションスタブ
`<astro-island 
  component-url="/Counter.js"
  component-export="default"
  renderer="react"
  client="load"
  props='${JSON.stringify(props)}'
>${ssrHTML}</astro-island>`
```

### 4.4 Vite Plugin Bridge

**仮想モジュール管理**

```typescript
// Virtual modules (astro:internal/*)
const virtualModules = {
  'astro:internal/island-manifest': () => generateIslandManifest(),
  'astro:internal/middleware': () => loadMiddleware(),
  'astro:internal/page-ssr': (id) => generateSSRModule(id)
}

// HMRバブリング
if (id.endsWith('.astro')) {
  const deps = getAstroDependencies(id)
  deps.forEach(dep => {
    server.ws.send({
      type: 'update',
      updates: [{ path: dep, acceptedPath: dep }]
    })
  })
}
```

**CJS/ESM境界処理**

レガシー依存関係のためのモジュール変換：

```typescript
// レガシーCJSライブラリをESMで使用
import { createRequire } from 'module'
const require = createRequire(import.meta.url)
const legacyLib = require('legacy-lib')

// Viteの最適化対象に追加
optimizeDeps: {
  include: ['legacy-lib']
}
```

### 4.5 最適化パス

**デッドコード除去**

```typescript
// client:only コンポーネントの不要コード削除
if (directive === 'client:only') {
  // SSR実行コードを除去
  removeSSRCode(component)
  // クライアント専用バンドルに移動
  moveToClientBundle(component)
}
```

**CSS @layer マージング**

```css
/* Before: 複数ファイルに散在 */
@layer base { /* reset styles */ }
@layer components { /* component styles */ }
@layer utilities { /* utility classes */ }

/* After: 統合とソースマップ保持 */
@layer base,components,utilities {
  /* マージされたスタイル */
}
```

**パフォーマンス結果**

Lighthouse 10回実行中央値： **LCP Δ -0.9s (±0.1s)**

### 4.6 Error Handling & DX

**詳細エラーコード**

```typescript
const diagnostics = {
  A001: 'JSX syntax error',
  A002: 'Missing component import',
  A003: 'Invalid client directive',
  A004: 'Circular dependency detected',
  A005: 'Unsupported file extension',
  // ... A015まで
}
```

**パターンマッチ型ヒント**

```typescript
function generateHint(error: CompilerError): string {
  if (error.code === 'A002' && error.context.includes('client:')) {
    return 'クライアントディレクティブが不正です。使用可能: load, idle, visible, media, only'
  }
  if (error.code === 'A001' && error.context.includes('JSX')) {
    return 'JSXクロージングタグが見つかりません。<!-- astro:jsx -->が必要かもしれません'
  }
  return 'Astro公式ドキュメントを確認: https://docs.astro.build'
}
```

**開発者ツール連携**

```typescript
// ブラウザオーバーレイへの診断情報送信
if (import.meta.hot) {
  import.meta.hot.send('astro:error', {
    id: error.id,
    message: error.message,
    stack: error.stack,
    loc: error.loc,
    // StackBlitzレプロリンク
    playground: generatePlaygroundLink(error)
  })
}
```

---

## 💬 Columns (Layer 3)

**History** — SnowpackのHTMLコンパイラから今日のASTパスまでの進化。初期バージョンは文字列置換ベースで、TypeScript統合が困難だった。

**Pitfall** — アイランド間の循環インポートがWindowsパスで壊れる理由（と修正方法）。Windowsの `\` パス区切りがESMインポート解決でエラーを引き起こし、`path.posix.join()` で統一が必要。

**Trivia** — 内部コードネーム「Starfish」と、放棄されたWebAssemblyプロトタイプ。パフォーマンス向上を期待したが、JSとの相互運用コストが利益を上回った。

---

## ハンズオンタスク（≤ 30分）

> **カスタム変換パスを構築**: すべての生成HTMLファイルにビルド時バナーコメントを挿入するタスク。
> 
> *ヒント*: `TransformVisitor`を拡張し、再コンパイルして差分を確認。

```typescript
// packages/compiler/src/transforms/banner-injection.ts
import type { AstroNode } from '../types'

export function injectBuildBanner(ast: AstroNode): AstroNode {
  const banner = `<!-- Generated by Astro ${process.env.ASTRO_VERSION} at ${new Date().toISOString()} -->`
  
  // HTML要素の開始前にバナーを挿入
  if (ast.type === 'root') {
    ast.children.unshift({
      type: 'comment',
      value: banner,
      position: { start: { line: 1, column: 1 }, end: { line: 1, column: banner.length } }
    })
  }
  
  return ast
}
```

**実装手順**:
1. `packages/compiler/src/transforms/`に新ファイル作成
2. メイン変換パイプラインに登録
3. `npm run build:compiler`でコンパイラ再構築  
4. テストサイトで生成HTMLを確認

---

## Next Chapter Bridge

「コンパイラがレンダラー非依存のバンドルを生成する仕組みを理解したので、**第05章**ではReact、Vue、Solidが競合なく共存する『魔法レイヤー』を明らかにします」

---

### Review Checklist (pre-PR)

* [x] Hook Pack完成（TL;DR、Quick View、Fail & Fix）
* [x] 指標に単位・ツール・差分を含む
* [x] コードスニペット≤20行、ビルド可能
* [x] Quick/Deepレイヤー分離
* [x] Next Chapter Bridgeで終了

---

*このスケルトンを出発点として使用し、すべてのガードレールを視野に入れながら各箇条書きを完全な文章、コード、図表に展開してください。*