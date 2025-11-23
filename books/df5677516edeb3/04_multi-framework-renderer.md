---
title: "マルチフレームワークレンダラーの統合"
---

# 第４章 マルチフレームワークレンダラーの統合

前章で、私たちはASTから静的なHTMLを生成するシンプルなビルダーを構築しました。これにより、`.astro`ファイルから初めて目に見える成果物を出力できましたが、生成されたHTMLはまだ完全に静的です。`<MyComponent />`のようなコンポーネントはHTMLコメントとして出力されるだけで、その中身は描画されません。Astroの真の力、すなわちReact、Vue、Svelteといった好みのUIフレームワークをシームレスに利用できる能力は、まだ発揮されていません。

この章の問いは、「React・Vue・Svelteを統一的に扱い、サーバーサイドでHTMLを生成するにはどうするか」です。この課題を解決するため、私たちは**統一レンダラーシステム**という抽象化レイヤーを設計・実装します。これは、各フレームワークが持つ固有のSSR（サーバーサイドレンダリング）APIの違いを吸収し、私たちのコンパイラから統一されたインターフェースで呼び出せるようにする仕組みです。この章を終えれば、私たちの自作フレームワークは、ついに複数のUIフレームワークを混在させて描画する能力を手に入れます。

## 課題：フレームワークごとに異なるSSR API

主要なUIフレームワークは、いずれもコンポーネントをサーバーサイドでHTML文字列に変換する機能を提供しています。しかし、そのAPIはそれぞれ異なります。

*   **React**: `react-dom/server`パッケージの`renderToString()`関数を使います。
    ```javascript
    import { renderToString } from 'react-dom/server';
    import MyReactComponent from './MyComponent.jsx';
    const html = renderToString(<MyReactComponent />);
    ```

*   **Vue**: `vue/server-renderer`パッケージの`renderToString()`関数を使います。
    ```javascript
    import { createSSRApp } from 'vue';
    import { renderToString } from 'vue/server-renderer';
    import MyVueComponent from './MyComponent.vue';
    const app = createSSRApp(MyVueComponent);
    const html = await renderToString(app);
    ```

*   **Svelte**: コンパイル後のコンポーネントが持つ`render()`メソッドを呼び出します。
    ```javascript
    import MySvelteComponent from './MyComponent.svelte';
    const { html } = MySvelteComponent.render({ prop: 'value' });
    ```

これらのAPIをビルドプロセスで直接扱おうとすると、`if`文や`switch`文による複雑な分岐が必要になり、コードの見通しは悪化します。新しいフレームワークを追加するたびに、ビルドシステムのコアロジックを修正しなければならず、拡張性も著しく損なわれます。

## 解決策：統一レンダラーインターフェースの設計

この問題を解決する鍵は、**抽象化**です。私たちは、すべてのフレームワークレンダラーが従うべき共通のルール、すなわち**インターフェース**を定義します。このインターフェースには、コンポーネントを描画するために最低限必要な機能を定義します。

```typescript
// packages/core/src/renderer.ts

export interface ComponentMetadata {
  componentPath: string; // コンポーネントのファイルパス
  props: Record<string, any>; // コンポーネントに渡すプロパティ
}

export interface Renderer {
  name: string; // レンダラーの名前 (e.g., 'react', 'vue')
  check(componentPath: string): boolean; // このレンダラーが担当するファイルか判定
  renderToString(metadata: ComponentMetadata): Promise<string>; // HTMLを生成
}
```

この`Renderer`インターフェースが、私たちのシステムの中心的な契約となります。`check`メソッドは、ファイルパス（例: `MyComponent.jsx`）を受け取り、その拡張子などから自身が処理すべきコンポーネントかどうかを判断します。`renderToString`メソッドは、コンポーネントのパスとプロパティ情報を受け取り、対応するフレームワークのSSR APIを呼び出してHTML文字列を返します。

## レンダラーの実装と登録

このインターフェースに従って、各フレームワーク用のレンダラーを実装してみましょう。

### Reactレンダラー

```typescript
// packages/renderer-react/src/index.ts
import type { Renderer, ComponentMetadata } from '@astro-lite/core';
import { renderToString } from 'react-dom/server';

export default {
  name: 'react',
  check: (componentPath) => componentPath.endsWith('.jsx') || componentPath.endsWith('.tsx'),
  renderToString: async ({ componentPath, props }) => {
    const { default: Component } = await import(componentPath);
    const element = <Component {...props} />;
    return renderToString(element);
  },
} satisfies Renderer;
```

### レンダラーレジストリ

次に、これらのレンダラーを管理する**レジストリ**を作成します。これは、利用可能なすべてのレンダラーを配列として保持するだけのシンプルなモジュールです。

```typescript
// packages/core/src/registry.ts
import type { Renderer } from './renderer';
import reactRenderer from '@astro-lite/renderer-react';
// import vueRenderer from '@astro-lite/renderer-vue'; // 同様に実装

export const renderers: Renderer[] = [
  reactRenderer,
  // vueRenderer,
];
```

## ビルドプロセスの更新

最後に、第3章で作成した`html-builder.ts`を更新し、このレンダラーシステムを組み込みます。`Component`ノードを処理する際に、レジストリから適切なレンダラーを探し出し、その`renderToString`メソッドを呼び出すように変更します。

```typescript
// packages/compiler/src/html-builder.ts
// ... (importなどを追記)
import { renderers } from '@astro-lite/core/registry';

async function renderNode(node: AstroNode): Promise<string> {
  switch (node.type) {
    // ... (element, textのケースは変更なし)

    case 'component': {
      // 適切なレンダラーを探す
      const renderer = renderers.find(r => r.check(node.tagName)); // tagNameにパスが入ると仮定

      if (!renderer) {
        return `<!-- Renderer not found for ${node.tagName} -->`;
      }

      // レンダラーに処理を委譲
      return await renderer.renderToString({
        componentPath: node.tagName, // 仮にtagNameをパスとして使用
        props: {}, // プロパティの受け渡しは次以降の課題
      });
    }

    // ... (その他のケース)
  }
}

export async function buildHtml(ast: AstroAST): Promise<string> {
  const results = await Promise.all(ast.children.map(renderNode));
  return results.join('');
}
```

> **注意:** 上記のコードは非同期処理を含むため、`renderNode`と`buildHtml`が`Promise`を返すように変更されています。また、簡単のため、コンポーネントのパスを`tagName`として扱い、プロパティの受け渡しを省略しています。これらの点は、後の章でビルドパイプライン全体を構築する際に、より洗練させていきます。

この変更により、ビルドプロセス自体はReactやVueの具体的なレンダリング方法を知る必要がなくなりました。新しいフレームワーク（例えばSvelte）に対応したい場合でも、`Renderer`インターフェースを実装した`renderer-svelte`パッケージを作成し、レジストリに登録するだけで済みます。コアロジックを変更する必要は一切ありません。これこそが、抽象化がもたらす強力な拡張性です。

## まとめと次のステップ

この章では、Astroの魔法の1つであるマルチフレームワーク対応の核心、**統一レンダラーシステム**を実装しました。フレームワーク固有のSSR APIの違いを吸収する`Renderer`インターフェースを設計し、ビルドプロセスから具体的な実装を分離することで、拡張性と保守性の高いアーキテクチャを実現しました。

私たちの自作フレームワークは、ついに静的なHTMLだけでなく、Reactのような外部フレームワークで記述されたコンポーネントを描画する能力を獲得しました。しかし、まだ道半ばです。

*   **ビルドパイプラインの欠如**: 現在は手動で各ファイルを処理していますが、開発サーバーでの自動リロード（HMR）や、本番用の最適化されたビルドを行う仕組みがありません。
*   **クライアントサイドのJavaScript**: サーバーで描画したコンポーネントに、ブラウザでインタラクティブ性を持たせる「ハイドレーション」の仕組みが全くありません。

次章では、これらの課題を解決するため、**Viteと連携した完全なビルドパイプラインの実装**に挑みます。Viteの強力なプラグイン機能を活用し、`.astro`ファイルの変換、開発サーバー、本番ビルドといった、モダンなフロントエンド開発に不可欠な機能を一気通貫で構築していきます。
