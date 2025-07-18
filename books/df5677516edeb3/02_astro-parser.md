---
title: ".astroファイルパーサーの実装"
---

# 第２章 .astroファイルパーサーの実装

前章では、Astroの根幹をなすIslands Architectureの設計思想を学び、なぜAstroが"コンテンツ中心・デフォルト高速"という哲学を選んだのかを理解しました。しかし、思想だけではブラウザに何も表示できません。具体的なウェブページを構築するには、まず`.astro`という独自形式のファイルをコンピュータが理解できる構造に変換する必要があります。

この章では、その変換プロセス、すなわち**パーサー**の実装に焦点を当てます。パーサーは、ソースコードを読み解き、その構造を抽象的なデータ構造である**抽象構文木（AST）**に変換するプログラムです。`.astro`ファイルは、フロントマター、HTMLライクな構文、そしてJavaScript式が混在する特殊な形式をしています。私たちの最初の課題は、この混在したテキストをいかにして正確に解析し、後続の処理で扱いやすいASTへと変換するか、その仕組みをゼロから構築することです。

## .astroファイルの構造を理解する

パーサーを実装する前に、対象となる`.astro`ファイルの構造を理解することが不可欠です。ファイルは主に3つの部分から構成されます。

1.  **Code Fence（フロントマター）**: ファイルの先頭に`---`で囲まれた部分。ここではサーバーサイドで実行されるJavaScriptコードを記述し、コンポーネントのロジックやデータの取得を行います。
2.  **Template（テンプレート）**: HTMLライクな構文でUIを記述する部分。静的なHTMLタグに加え、コンポーネントやJavaScript式を埋め込むことができます。
3.  **Style（スタイル）**: `<style>`タグ内に記述されるCSS。デフォルトでスコープがコンポーネント内に限定されるという特徴があります。

```astro
---
// 1. Code Fence (Frontmatter)
import MyComponent from '../components/MyComponent.astro';
const title = 'Astro';
---

<!-- 2. Template -->
<html lang="ja">
<head>
  <title>{title}</title>
</head>
<body>
  <h1>ようこそ、{title}へ！</h1>
  <MyComponent client:load />
</body>
</html>

<style>
/* 3. Style */
h1 {
  color: orange;
}
</style>
```

この構造を解析するには、単純なHTMLパーサーでは不十分です。Code FenceとTemplateを明確に分離し、Template内に埋め込まれたJavaScript式 `{title}` を識別する能力が求められます。

## パーサーの設計選択：正規表現 vs ステートマシン

パーサーの実装アプローチにはいくつか選択肢がありますが、ここでは代表的な2つを比較します。

*   **正規表現ベース**:
    *   **長所**: 比較的シンプルで、特定のパターン（例えばCode Fence）を素早く切り出すのに適しています。
    *   **短所**: ネストした構造や複雑な構文規則を扱うのが苦手で、メンテナンス性が低くなりがちです。HTMLのような複雑な言語の解析には限界があります。

*   **ステートマシンベース**:
    *   **長所**: 文字を1つずつ読みながら「現在の状態（例：HTMLタグ内、属性内、テキスト内）」を管理するため、複雑な構文規則にも柔軟に対応できます。拡張性も高いです。
    *   **短所**: 実装が正規表現ベースより複雑になります。

Astroの構文はHTMLを基礎としながらも独自の拡張を含んでいるため、堅牢性と拡張性を重視し、**ステートマシンを基本としたアプローチ**を採用するのが賢明です。ただし、Code Fenceの切り出しのような単純な部分には正規表現を補助的に利用することで、両者の利点を活かすことができます。

## 最小限のパーサーを実装する

それでは、実際にパーサーの実装に取り掛かりましょう。私たちの目標は、`.astro`ファイルのソース文字列を受け取り、それをASTに変換する`parse`関数を作ることです。

### Step 1: Code Fenceの分離

最初のステップとして、最も単純な部分であるCode Fenceを正規表現で切り出します。

```typescript
// packages/compiler/src/parse.ts

export interface AstroAST {
  type: 'root';
  script: string | null;
  html: string;
}

const FRONTMATTER_REGEX = /---\s*([\s\S]*?)\s*---/;

export function parse(source: string): AstroAST {
  const match = source.match(FRONTMATTER_REGEX);

  if (match) {
    const script = match[1];
    const html = source.slice(match[0].length);
    return { type: 'root', script, html };
  }

  // フロントマターがない場合
  return { type: 'root', script: null, html: source };
}
```

このコードは、`---`で囲まれた部分を`script`として、それ以降を`html`として単純に分離します。これで、コンパイラの最初の部品が完成しました。しかし、この状態ではTemplate部分がただの文字列に過ぎず、その中身の構造は全く理解できていません。

> **落とし穴:** この正規表現は非常にシンプルであり、ソースコード内に`---`という文字列が偶然含まれていた場合に誤動作する可能性があります。プロダクションレベルのパーサーでは、より厳密な状態管理が必要になりますが、ここではまず最小限の動作を優先します。

### Step 2: HTML部分のトークナイズ

次に、分離したHTML文字列を意味のある最小単位、すなわち**トークン**に分割する**トークナイザー**を実装します。トークンとは、「開始タグ」「テキスト」「終了タグ」といった構文上の要素です。

ここでは、簡単のため、非常に単純なHTMLパーサーライブラリ `html-parser-lite` を使うことにします。（本来は自作しますが、本章の主題から逸れるためライブラリで代用します）

```bash
# 依存関係の追加
pnpm add html-parser-lite -w
```

```typescript
// packages/compiler/src/parse.ts
import { parse as parseHtml, Node } from 'html-parser-lite';

export interface AstroAST {
  type: 'root';
  script: string | null;
  children: Node[]; // html: string から変更
}

// ... 以前のコード

export function parse(source: string): AstroAST {
  const match = source.match(FRONTMATTER_REGEX);

  if (match) {
    const script = match[1];
    const htmlSource = source.slice(match[0].length);
    const children = parseHtml(htmlSource);
    return { type: 'root', script, children };
  }

  const children = parseHtml(source);
  return { type: 'root', script: null, children };
}
```

`parseHtml`関数は、HTML文字列を解析し、タグやテキストを表すオブジェクトの配列（`Node[]`）を返します。これで、Template部分が単なる文字列ではなく、構造化されたデータになりました。

しかし、このパーサーはまだAstro特有の構文を理解できません。例えば、`{title}`のようなJavaScript式や、`<MyComponent />`のようなコンポーネントタグは、通常のHTMLタグとして誤って解釈されてしまいます。

### Step 3: Astro特有の構文をASTに組み込む

最終ステップとして、標準のHTML ASTを走査し、Astro特有の構文を私たちのASTノードに変換する処理を追加します。

*   **JavaScript式 (`{...}`)**: テキストノードの中に`{`と`}`で囲まれた部分を見つけたら、それを`Expression`ノードとして表現します。
*   **コンポーネントタグ**: タグ名の先頭が大文字で始まっている場合、それは標準のHTMLタグではなくAstroコンポーネントであると判断し、`Component`ノードとして扱います。

この変換処理を`transform`関数として実装します。

```typescript
// packages/compiler/src/transform.ts
import { Node } from 'html-parser-lite';

// AstroのASTノード型を定義
export type AstroNode =
  | { type: 'element'; tagName: string; children: AstroNode[] }
  | { type: 'text'; value: string }
  | { type: 'expression'; value: string }
  | { type: 'component'; tagName: string; children: AstroNode[] };

function isComponent(tagName: string): boolean {
  return tagName[0] === tagName[0].toUpperCase();
}

export function transform(nodes: Node[]): AstroNode[] {
  return nodes.map(node => {
    if (node.type === 'tag') {
      const children = transform(node.children);
      if (isComponent(node.name)) {
        return { type: 'component', tagName: node.name, children };
      }
      return { type: 'element', tagName: node.name, children };
    }
    if (node.type === 'text') {
      // ここでは簡単のため、式({..})の変換はまだ実装しない
      return { type: 'text', value: node.content };
    }
    // 他のノードタイプは無視
    return null;
  }).filter(Boolean) as AstroNode[];
}
```

そして、`parse`関数を更新して、この`transform`処理を組み込みます。

```typescript
// packages/compiler/src/parse.ts
// ...
import { transform, AstroNode } from './transform';

export interface AstroAST {
  type: 'root';
  script: string | null;
  children: AstroNode[];
}

export function parse(source: string): AstroAST {
  const match = source.match(FRONTMATTER_REGEX);
  const htmlSource = match ? source.slice(match[0].length) : source;
  const script = match ? match[1] : null;

  const initialAST = parseHtml(htmlSource);
  const children = transform(initialAST);

  return { type: 'root', script, children };
}
```

これで、`.astro`ファイルを解析し、フロントマター、HTML要素、そしてAstroコンポーネントを区別できる、よりリッチなASTを生成するパーサーの原型が完成しました。

## まとめと次のステップ

この章では、Astroコンパイラの心臓部であるパーサーの実装に挑戦しました。正規表現とステートマシンベースのパーサーを比較検討し、`.astro`ファイルからCode Fenceを分離し、HTML部分をトークナイズしてASTへと変換する一連のプロセスを段階的に実装しました。

現在のパーサーは、Astroコンポーネントを識別できますが、まだ多くの課題が残されています。

*   **JavaScript式の未対応**: `{title}`のような式は、まだプレーンなテキストとして扱われています。
*   **属性の解析**: `<div class="foo">`のような属性が完全に無視されています。
*   **ディレクティブの未対応**: `client:load`のようなAstro特有のディレティブを理解する仕組みがありません。

これらの課題は、ASTをさらに加工する**トランスフォーマー**や、最終的な出力を生成する**コードジェネレーター**の役割となります。

次章では、この章で生成したASTを元に、実際にブラウザで表示可能な純粋なHTMLを生成する**シンプルなHTMLビルダー**を構築します。これにより、私たちは初めて、自作のツールチェーンで`.astro`ファイルから静的なウェブページを生成する体験をすることになります。ASTがどのようにして具体的な成果物へと変わっていくのか、その過程を見ていきましょう。