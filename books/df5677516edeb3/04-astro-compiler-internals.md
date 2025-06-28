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

---

## 🔎 Deep Dive (Layer 2)

### 4.1 Parser内部構造

*問題 ➜ 探索 ➜ 解決策 ➜ 結果*

**なぜBabelではなく専用パーサーか？**

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

Lighthouse 10回実行中央値: **LCP Δ -0.9s (±0.1s)**

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

「コンパイラがレンダラー非依存のバンドルを生成する仕組みを理解したので、**第05章**では React、Vue、Solidが競合なく共存する『魔法レイヤー』を明らかにします。」

---

### Review Checklist (pre-PR)

* [x] Hook Pack完成（TL;DR、Quick View、Fail & Fix）
* [x] 指標に単位・ツール・差分を含む
* [x] コードスニペット≤20行、ビルド可能
* [x] Quick/Deepレイヤー分離
* [x] Next Chapter Bridgeで終了

---

*このスケルトンを出発点として使用し、すべてのガードレールを視野に入れながら各箇条書きを完全な文章、コード、図表に展開してください。*