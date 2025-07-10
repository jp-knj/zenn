## Astroを"再実装"する教科書

「もしAstroを自作するとしたら？」—この問いをガイドに、コンパイラ、アイランドアーキテクチャ、レンダラーの核心を、手を動かしながら再実装していく。フレームワークの内部構造を深く理解し、思考実験を骨格に据えた実践的なリバース・エンジニアリング

- コンセプト:
  - 「再実装」の視点: Astroの機能を単に「使う」だけでなく、「なぜそのように作られているのか」「もし自分で作るとしたらどう設計するか」というフレームワーク開発者の視点から深く掘り下げます。
  - 没入感のある学習体験: 各章が、仮想のフレームワークを開発していく過程として描かれ、読者がその設計・実装の思考プロセスを追体験できるような物語性を持たせます。
  - 深い技術的理解: コンパイラ、アイランドアーキテクチャ、レンダラー、Vite連携といったAstroの核心技術を、一次情報に基づいた具体的なコード例や事例を交えながら詳細に解説します。
  - 実践的なアプローチ: 各章で提示される課題や演習を通じて、読者が実際に手を動かし、知識を定着させることを重視します。
  - 高品質な文章: 箇条書きを避け、流れるような散文で、客観的かつ想像しやすい内容を目指します。

## 第１部　思想と設計の原点

### 第１章　Astroの設計思想とIslands Architecture

問い：なぜAstroは"必要なJavaScriptだけを届ける"という発想に至り、どのような設計で実現したのか

JavaScript肥大化の歴史をひもとき、SPAとSSRの進化が抱えた負債を整理する。Jason Millerによる Islands Architectureの提唱がいかに議論を活性化し、Astroが「コンテンツ中心・デフォルト高速」という哲学を選び取ったのかを、当時のコミュニティ動向と照らしながら描く。"海と島"にページを分けることで、JavaScriptをページ単位で抱える従来手法との差異を実測ベンチマークと概念図で検証し、通信量・実行時メモリ・INPの推移を数値で示す。React Server ComponentsやNuxt Islandなど近縁アプローチと並べ、設計思想の分岐と優位性を俯瞰する。

## 第２部　コア技術の深層

### 第２章　ビルドパイプライン：静的HTML生成

問い：.astroファイルはどのように解析・変換され、どこまでJavaScriptを削減できるのか

ここから実装へ移る。`.astro`ファイルがコンパイラに読み込まれ、抽象構文木へ変換され、フレームワークごとのレンダラーがサーバーサイドでHTMLを生成し、最後にViteがtree shakingとcode splittingで不要なバイトを削ぎ落とす。その工程をTypeScriptの行単位まで追跡する。フェーズごとに削減できたファイルサイズやビルド時間を計測し、前章の思想が具体的に形になる。

**実装演習**: 章末で簡易.astroパーサーの実装（100行程度）を行い、基本的な構文解析の仕組みを体験する。

### 第３章　コンパイラ内部構造の探訪

問い：混在するHTML／JSX／TypeScriptを、Astroコンパイラはどう高速にさばくのか

ストリーミング・パーサーとインクリメンタルAST、型情報の逆参照を駆使したエラー検出を分解し、１ファイルを変換するたびに何がキャッシュされ、何が再計算されるのかを明らかにする。

**実装演習**: 基本的なトランスフォーマーを実装し、ASTからHTMLへの変換プロセスを実践する。

### 第４章　ランタイム：選択的ハイドレーションの実際

問い：ブラウザは<astro-island>をいつ、どのようにインタラクティブへ変貌させるのか

Astroが仕込んだ軽量スクリプトがマニフェストをもとに島を特定し、client:*戦略ごとにJSを遅延ロードする様子をトレースする。IntersectionObserverがトリガを司る場面や、イベント再結合で発生する落とし穴と回避策も示す。

**実装演習**: client:visibleの最小実装を通じて、遅延ハイドレーションの核心を理解する。

### 第５章　マルチフレームワーク統合技術

問い：React・Vue・Svelteを１ページで共存させる設計はどう成り立つのか

統一レンダラーAPIと各フレームワーク固有のSSR手続きの橋渡しロジックを紐解き、島ごとの状態共有やイベント境界の扱いまで踏み込む。

### 第６章　ハイドレーション戦略とパフォーマンス最適化

問い：`client:*`戦略を切り替えると実測INPはどこまで改善できるのか

load・visible・idle・media各戦略の内部実装を読み解き、ベンチマークサイトでCore Web Vitalsがどう変化するかを検証する。遅延実行の副作用として発生するUXギャップへの対応策も探る。

### 第７章　Viteプラグインアーキテクチャ

問い：AstroはViteをどう拡張して.astro専用ローダーとisland最適化を成立させているのか

開発サーバーとRollupビルドの二面構造を横断し、仮想モジュール、HMR、並列バンドルが.astroの開発体験をどのように支えているかを解剖する。

## 第３部　実践と応用、そして未来

### 第８章　Content Collectionsと型安全なコンテンツ管理

問い：フロントマターの型不整合をどのように解消し、Markdownを型安全に扱うか

Zodスキーマで宣言した型をもとにビルド時検証とTypeScript型生成を行う仕組みを実装し、コンテンツ間の参照整合性まで保証するワークフローを提示する。

### 第９章　パフォーマンス計測と最適化

問い：Astroサイトのボトルネックをどう特定し、どの順序で改善するか

ラボ計測とRUMを組み合わせ、LCP・CLS・INPを継続監視するCIパイプラインを構築。最適化実験の結果を数値で検証し、失敗例も共有する。

### 第１０章　Astroの拡張とエコシステム

問い：独自インテグレーションとサーバーエンドポイントでAstroをどこまで拡張できるか

Integration APIとBuild Hooksを駆使し、APIルートやCMS連携、分析ツール挿入など高度なユースケースを構築。Astroのロードマップとコミュニティ動向も展望し、読者自身の拡張アイデアを刺激する。

---

## 読後の成果物

最終的に読者はmonorepo形式の **astro-lite** を完成させる。`pnpm dev`で`examples/blog`を立ち上げれば、Islands Architectureと型安全Content Collectionsを兼ね備えたHMR環境が動作する。各章の実装演習が積み重なり、この成果物へと結実する。

```text
astro-lite/
├─ .gitignore
├─ package.json
├─ tsconfig.base.json
├─ turbo.json
├─ astro.config.mjs
│
├─ packages/
│  ├─ core/
│  │  ├─ src/
│  │  │  ├─ index.ts
│  │  │  └─ logger.ts
│  │  └─ package.json
│  │
│  ├─ compiler/                    # 第２章・第３章で実装
│  │  ├─ src/
│  │  │  ├─ parse.ts              # 第２章：パーサー実装
│  │  │  ├─ transform.ts          # 第３章：トランスフォーマー実装
│  │  │  ├─ serializer.ts
│  │  │  └─ codegen.ts
│  │  └─ package.json
│  │
│  ├─ vite-plugin/                # 第７章で実装
│  │  ├─ src/
│  │  │  ├─ load-astro.ts
│  │  │  ├─ island-plugin.ts
│  │  │  └─ content-plugin.ts
│  │  └─ package.json
│  │
│  ├─ runtime/                    # 第４章で実装
│  │  ├─ src/
│  │  │  ├─ index.ts
│  │  │  ├─ manifest.ts
│  │  │  └─ strategies/          # 第６章で拡張
│  │  │     ├─ load.ts
│  │  │     ├─ visible.ts
│  │  │     └─ idle.ts
│  │  └─ package.json
│  │
│  ├─ renderer-react/             # 第５章で実装
│  │  └─ src/index.ts
│  ├─ renderer-vue/
│  │  └─ src/index.ts
│  ├─ renderer-svelte/
│  │  └─ src/index.ts
│  │
│  ├─ content/                    # 第８章で実装
│  │  ├─ src/
│  │  │  ├─ index.ts
│  │  │  ├─ loaders/
│  │  │  │  ├─ md-loader.ts
│  │  │  │  └─ json-loader.ts
│  │  │  ├─ schema/
│  │  │  │  └─ zod-helpers.ts
│  │  │  ├─ generator/
│  │  │  │  ├─ manifest.ts
│  │  │  │  └─ dts-gen.ts
│  │  │  ├─ vite-plugin.ts
│  │  │  └─ runtime.ts
│  │  └─ package.json
│  │
│  └─ types/
│     ├─ astro-lite-content.d.ts
│     └─ package.json
│
└─ examples/
   ├─ blog/
   │  ├─ src/
   │  │  ├─ content/
   │  │  │  ├─ blog/welcome.md
   │  │  │  └─ authors/jp-knj.json
   │  │  ├─ pages/
   │  │  │  └─ blog/index.astro
   │  │  └─ content.config.ts
   │  ├─ astro.config.mjs
   │  └─ package.json
   └─ benchmark/
       └─ lighthouseci.config.mjs
```