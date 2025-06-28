---
title: "マルチフレームワーク統合技術"
---

> ### 章の読み方ガイド
> - **⚡Quick View** だけ追えば 5 分で流れが掴める
> - **🔎Deep Dive** を読むとコードや実装差まで理解できる
> - **💬Column** は 1 ページ以内の逸話・数字・失敗談スパイス

## 0. イントロ ─ “混在フレームワーク”3つの罠
### 0.1 LCP が **+1.3 s** 悪化した実例（数字グラフ） ⚡
### 0.2 hydration mismatch (#8178) で UI 崩壊した失敗談 💬
### 0.3 本章で到達するゴール 🔎

## 1. 統一レンダラーシステムを設計する
### 1.1 共通インターフェース `check / renderToStaticMarkup / hydrate` ⚡
### 1.2 フレームワーク自動判定アルゴリズム 🔎
### 1.3 props 直列化と型安全保証 🔎
<aside class="column">
**Inside Slack 💬**  
Islands アイデア誕生の Slack ログ（1 ページ）
</aside>

## 2. レンダラー実装を“差分表”で俯瞰する ⚡
| FW | `check()` 特徴 | SSR API | Hydration API | 注目ポイント |
|----|---------------|---------|---------------|--------------|
| React  | `$$typeof`        | `renderToString`   | `hydrateRoot`       | Concurrent, Suspense |
| Vue    | `__vueOptions__`  | `@vue/server-renderer` | `createSSRApp`  | Teleport, Composition API |
| Svelte | `render` func     | compile-time       | `new Component`     | 0-runtime, scoped CSS |
| Solid  | fn signature      | `renderToStringAsync` | `hydrate`      | No VDOM, fine-grained |

### 2.1 各レンダラーの “ここが違う” をコードで比較 🔎

## 3. 異種フレームワーク協調の内部実装
### 3.1 ビルド時レンダラー検出フロー図 ⚡
### 3.2 SSR パイプラインでの呼び出し順 🔎
### 3.3 `<astro-island>` ラッパー生成 & メタデータ埋込み 🔎

## 4. 状態と通信 ― “島”をつなぐ
### 4.1 アイランド間分離アーキテクチャ ⚡
### 4.2 カスタムイベント駆動通信 🔎
### 4.3 永続ストレージ / URL シンクによる状態共有 🔎
<aside class="column">
**Performance Nugget 💬**  
`client:visible` だけで JS DL を **−46%** 削減したケース
</aside>

## 5. ハンズオン🧪 — 自作レンダラーを 30 分で
1. `astro add my-lib-renderer` の雛形を生成
2. `check()` にシグネチャ検出を実装
3. `renderToStaticMarkup()` で “Hello” SSR
4. PRを作成しCIが緑になるまで

✅ **チェックリスト** 付き（15–30 min）

## 6. まとめ → 06 章「ハイドレーション戦略」へ
- 学び3行まとめ
- “アイランド生成” が **ハイドレーション戦略** とどう連携するかを予告
