---
title: "パフォーマンス計測と最適化"
---

## 【この章で学ぶこと】

これまで、Astroのアーキテクチャがいかにパフォーマンスを重視して設計されているかを学んできました。この章では、その理論が実際のWebサイトでどのような結果として現れるのかを、具体的な数値と共に検証します。再現性のある"ラボ計測"と、実ユーザー環境での"フィールドデータ"という両面からアプローチし、Astroサイトの性能を客観的に評価する方法と、さらなる最適化のためのヒントを探ります。

## 1. 【課題】パフォーマンスは"測る"まで分からない

"Astroは速い"と聞かされても、それが具体的にどの程度で、どのような指標で優れているのかを理解しなければ、その価値を正しく評価することはできません。また、パフォーマンスは一度きりの最適化で終わるものではなく、新しい機能を追加したり、コンテンツが増えたりするたびに変化しうる、継続的な監視が必要なものです。例えば、あるメディアサイトがGatsbyからAstroへ移行したケースでは、ヒーロー画像がReactコンポーネント内で`useEffect`を使って動的に読み込まれていたことが、LCP（Largest Contentful Paint）を3.8秒まで遅延させるボトルネックとなっていました。このような問題は、実際に計測して初めて明らかになることがほとんどです。

## 2. 【実践】計測環境の構築とCore Web Vitalsの分析

パフォーマンス計測の再現性を担保するためには、環境を厳密に定義することが重要です。ラボ計測では、GoogleのLighthouse CLIツールを用い、WebPageTestが提供する"3G-Slow"のような低速回線プロファイルをシミュレートして、複数回測定の平均値を取ることが推奨されます。一方で、より現実的な指標を得るためには、Google Analytics 4などのRUM（Real User Monitoring）ツールから得られるフィールドデータを分析し、ユーザーの75パーセンタイル値（p75）を評価することが不可欠です。

Googleが重要視するCore Web Vitalsの3指標、すなわちLCP、CLS、そして2024年3月にFIDから置き換わったINP（Interaction to Next Paint）に注目しましょう。Astroは、サーバーサイドでHTMLを生成し、不要なJavaScriptを徹底的に排除するため、ブラウザがコンテンツを素早く描画でき、LCPの改善に大きく貢献します。Firebaseの公式サイトがAstroへ移行した際のケーススタディでは、Lighthouseのパフォーマンススコアが50点台から90点台へと劇的に向上したことが報告されており、これはAstroのアーキテクチャの優位性を示す強力な一次情報です。

また、INPに関しても、Astroは有利な特性を持っています。SPAでは初期ロード時に全コンポーネントのイベントハンドラが読み込まれ、メインスレッドを逼迫させがちですが、Astroではインタラクティブなアイランドのみがハイドレーション対象となります。このため、ボタンクリック等のインタラクションに対する応答性が高く、INPは良好な数値を維持しやすいのです。

## 3. 【実践】Performance Observer APIでWeb Vitalsを計測する

Lighthouseのような総合的なツールも強力ですが、特定のパフォーマンス指標を継続的に監視したい場合、ブラウザに組み込まれている`PerformanceObserver` APIが非常に役立ちます。これを使って、Core Web Vitalsの各指標をコンソールに出力する簡単なスクリプトを作成してみましょう。

このコードは、どのウェブサイトでもブラウザの開発者ツールのコンソールに貼り付けて実行できます。

```javascript
// ブラウザのコンソールで実行するコード

console.log('Performance Observerを起動します。')

const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    // CLS (Cumulative Layout Shift)
    if (entry.entryType === 'layout-shift') {
      console.log(`CLS: ${entry.value}`)
    }
    // LCP (Largest Contentful Paint)
    if (entry.entryType === 'largest-contentful-paint') {
      console.log(`LCP: ${entry.startTime.toFixed(2)}ms`)
    }
    // INP (Interaction to Next Paint) の近似値として Long Animation Frames を監視
    if (entry.entryType === 'long-animation-frame') {
      console.log(`Long Animation Frame (INPのヒント): ${entry.duration.toFixed(2)}ms`)
    }
  }
})

// 監視対象のパフォーマンス指標を指定
observer.observe({
  type: ['layout-shift', 'largest-contentful-paint', 'long-animation-frame'],
  buffered: true, // 過去のイベントも取得
})

console.log('LCP, CLS, Long Animation Frame (INPのヒント) の監視を開始しました。')
```

このスクリプトを実行した状態でページを操作（スクロールやクリック）すると、レイアウトのズレ（CLS）や、最も大きなコンテンツの表示時間（LCP）、そしてインタラクションへの応答性のヒントとなる長い処理（Long Animation Frame）が発生するたびに、その値がコンソールに出力されます。

例えば、ハイドレーション戦略の章で試したように、全コンポーネントを`client:load`にした場合と、`client:visible`などに最適化した場合とで、このスクリプトを実行してLCPやLong Animation Frameの数値を比較すれば、Astroの最適化がブラウザレベルでどのような影響を与えているかを直接的に観測できます。

## 4. 【考察】バンドルサイズとアーキテクチャの関連性

Astroサイトのパフォーマンスの根源は、その圧倒的に小さなバンドルサイズにあります。Next.jsのようなSPAフレームワークは、ページの描画、ルーティング、状態管理を行うための"ランタイム"をクライアント側で必要とするため、同等のページでも数十から数百キロバイトのJavaScriptを要求します。対して、Astroはデフォルトでは一切のクライアントサイドランタイムを持たず、HTMLを直接配信するため、JavaScriptの初期ロードサイズを数キロバイトにまで抑えることが可能です。この差が、特にモバイルデバイスや低速回線環境下での体感速度に決定的な違いを生むのです。

Astroでビルド時に画像URLを取得し、`<img>`タグを直接HTMLに埋め込むように修正するだけで、LCPが2秒以上短縮されたという事例もあります。これは、ブラウザがHTMLを解析した時点で画像の存在を検知し、すぐに読み込みを開始できるためです。JavaScriptの実行を待つ必要がないという、サーバーファーストアーキテクチャの単純かつ強力な利点を示しています。

## 5. 【まとめと次章へ】

本章では、Astroへの移行がLCPやバンドルサイズに与える定量的な効果を、再現性のある手順と共に示しました。また、INPを含むCore Web Vitalsの全項目が良好な結果を示す技術的背景を解明しました。パフォーマンスは一度きりの最適化で終わるものではなく、継続的な計測と改善が不可欠です。

最終章となる次章では、ここで得た知見を応用し、Astroの機能をさらに拡張する方法を探ります。APIルートの作成、カスタムインテグレーションの開発など、Astroを静的サイトジェネレーターの枠を超えた、より強力なアプリケーションプラットフォームとして活用する技術を解説します。
