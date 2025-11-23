---
title: "シンプルなHTMLビルダーの構築"
---

# 第３章 シンプルなHTMLビルダーの構築

前章で、私たちは`.astro`ファイルのソースコードを解析し、その構造を表現する抽象構文木（AST）へと変換するパーサーを実装しました。これはコンパイラ開発における重要な一歩です。しかし、ASTはあくまでコンピュータが理解しやすい中間表現に過ぎず、それ自体をブラウザで表示することはできません。私たちの最終目標は、ユーザーが見ることのできるウェブページを生成することです。

この章の問いは、「ASTから純粋なHTMLをどのように生成し、最初の静的ページを作り出すか」です。このプロセスを担うのが、**コードジェネレーター**、あるいは今回の目的に特化して**HTMLビルダー**と呼ばれるコンポーネントです。私たちは、前章で作成したASTを走査（トラバース）し、それを元にHTMLの文字列を組み立てるロジックを実装します。この章を終える頃には、私たちの自作ツールチェーンは初めて、`.astro`ファイルから目に見える成果物を生み出す能力を持つことになります。

## ASTからHTMLへの変換戦略

ASTは木のような階層構造を持っています。一方、HTMLは線形的なテキストファイルです。この構造の違いを乗り越える鍵は、**再帰的なASTの走査**にあります。

基本的な戦略は以下の通りです。

1.  ASTのルートノードから出発します。
2.  現在のノードの種類（`element`, `text`, `component`など）を判別します。
3.  ノードの種類に応じたHTML文字列を生成します。
    *   `element`ノードの場合、開始タグを生成し、その子ノード群に対して再帰的にこのプロセスを適用し、最後に終了タグを追加します。
    *   `text`ノードの場合、その内容をそのまま文字列として返します。
4.  すべてのノードの処理が終わるまでこれを繰り返します。

このアプローチにより、木構造を深さ優先で探索しながら、対応するHTML文字列を順番に組み立てていくことができます。

## HTMLビルダーの実装

それでは、この戦略をコードに落とし込んでいきましょう。`packages/compiler/src/html-builder.ts`という新しいファイルを作成し、ASTを受け取ってHTML文字列を返す`buildHtml`関数を実装します。

### Step 1: ビルダーの骨格

まず、ASTの各ノードを処理する中心的な関数`renderNode`と、それを呼び出すエントリーポイント`buildHtml`の骨格を定義します。

```typescript
// packages/compiler/src/html-builder.ts
import { AstroAST, AstroNode } from './transform';

function renderNode(node: AstroNode): string {
  switch (node.type) {
    case 'element':
      // TODO: 実装
      return '';
    case 'component':
      // TODO: 実装
      return '';
    case 'text':
      // TODO: 実装
      return '';
    case 'expression':
      // TODO: 実装
      return '';
    default:
      return '';
  }
}

export function buildHtml(ast: AstroAST): string {
  return ast.children.map(renderNode).join('');
}
```

`buildHtml`関数は、ASTのトップレベルの子ノード（`children`）をそれぞれ`renderNode`で処理し、その結果を結合して最終的なHTMLを生成します。

### Step 2: 各ノードタイプの処理

次に、`switch`文の中身を一つずつ実装していきます。

**Textノード**: 最も単純なケースです。テキストノードは、その値をそのまま返すだけです。

```typescript
// in renderNode function
case 'text':
  return node.value;
```

**Elementノード**: HTML要素を表現します。開始タグ、子ノードのレンダリング結果、そして終了タグを結合して返します。

```typescript
// in renderNode function
case 'element': {
  const childrenHtml = node.children.map(renderNode).join('');
  return `<${node.tagName}>${childrenHtml}</${node.tagName}>`;
}
```

> **補足:** この実装はまだ属性（`class`や`id`など）を扱っていません。属性の処理は後の章でトランスフォーマーを拡張する際に実装します。今はまず、基本的な構造を生成することに集中します。

**ComponentノードとExpressionノード**: これらはAstro特有のノードであり、純粋なHTMLには直接対応するものがありません。現時点では、これらをどのように扱うべきでしょうか？

*   **Componentノード**: 最終的には対応するフレームワーク（ReactやVueなど）のレンダラーを呼び出してHTMLを生成する必要がありますが、その仕組みはまだありません。ここでは、開発者がデバッグしやすいように、コンポーネントが存在したことを示すHTMLコメントとして出力するのが良いでしょう。
*   **Expressionノード**: `{title}`のようなJavaScript式は、サーバーサイドで評価され、その結果が埋め込まれるべきです。しかし、その実行エンジンもまだ存在しません。今は空文字列として扱い、何も出力しないことにします。

```typescript
// in renderNode function
case 'component':
  // どのコンポーネントが使われたかを示すコメントを出力
  return `<!-- <${node.tagName} /> -->`;

case 'expression':
  // サーバーサイドの実行環境がないため、今は何も出力しない
  return '';
```

### Step 3: 全体の統合

すべてを統合すると、`html-builder.ts`は以下のようになります。

```typescript
// packages/compiler/src/html-builder.ts
import { AstroAST, AstroNode } from './transform';

function renderNode(node: AstroNode): string {
  switch (node.type) {
    case 'element': {
      const childrenHtml = node.children.map(renderNode).join('');
      // 自己完結タグ（例: <meta>, <img>）の対応は省略
      return `<${node.tagName}>${childrenHtml}</${node.tagName}>`;
    }
    case 'component':
      return `<!-- <${node.tagName} /> -->`;
    case 'text':
      return node.value;
    case 'expression':
      return '';
    default:
      // 知らないノードは無視
      return '';
  }
}

export function buildHtml(ast: AstroAST): string {
  return ast.children.map(renderNode).join('');
}
```

これで、ASTから静的なHTMLを生成する最小限のビルダーが完成しました。パーサーとビルダーを組み合わせれば、`.astro`ファイルからHTMLへの変換パイプラインが実現します。

```typescript
// 仮想的なビルドスクリプト
import { parse } from './parse';
import { buildHtml } from './html-builder';
import * as fs from 'fs';

const source = fs.readFileSync('./src/pages/index.astro', 'utf-8');
const ast = parse(source);
const html = buildHtml(ast);

console.log(html);
```

このスクリプトを実行すれば、コンソールに生成されたHTMLが出力されるはずです。私たちはついに、コードからウェブページへの変換を達成したのです。

## まとめと次のステップ

本章では、前章で作成したASTを入力として受け取り、静的なHTML文字列を生成するHTMLビルダーを実装しました。再帰的なAST走査という手法を用いて、階層的なデータ構造を線形的なテキストへと変換するプロセスを学びました。これにより、私たちのコンパイラは初めて、目に見える成果物を生み出すことができるようになりました。

しかし、生成されたHTMLはまだ"静的"の域を出ません。現状のビルダーには、いくつかの大きな課題が残されています。

*   **コンポーネントが描画されない**: `<MyComponent />`は、実際のUIではなくHTMLコメントとして出力されるだけです。
*   **動的なデータが反映されない**: フロントマターで定義した変数や、`{title}`のような式は完全に無視されています。
*   **インタラクティビティの欠如**: クライアントサイドで実行されるべきJavaScriptを扱う仕組みがありません。

これらの課題を解決するには、単なるHTMLビルダーを超え、外部のレンダリングエンジンと連携し、サーバーサイドでコンポーネントを描画する仕組みが必要です。

次章では、この課題に取り組むため、**マルチフレームワークレンダラーの統合**に挑戦します。ReactやVueといった異なるUIフレームワークのSSR（サーバーサイドレンダリング）機能を抽象化し、私たちのコンパイラから呼び出せるようにする仕組みを設計・実装していきます。これにより、Astroの大きな特徴である「好きなUIフレームワークを使える」という機能の核心に迫ります。
