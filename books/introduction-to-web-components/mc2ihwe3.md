---
title: "Web Components の歴史"
---
## はじめに
どのようなプロセスで Web Components が生まれたのか、その歴史を振り返る。

XMLHttpRequest が標準化され、技術的にはもう特別なライブラリがAjaxには必要なくなりました。
私たちの中には、ライブラリを捨ててプラットフォームを直接使用し始めた人もいました。
しかし、時間が経つにつれて、私たちは生の XMLHttpRequest API が不十分である。
複雑さを管理しギャップを埋めるためにマイクロライブラリを使用し始めました。
さらに時間が経過し、プラットフォームは再び成長し、Promise と Fetch API を導入しました。

それに応じて、私たちは再びライブラリを捨ててプラットフォームに戻りました。

説明したことはパターンであることを示したいからです。
これは、すべての産業で様々な形で繰り返し起こります。私はそれをこのように要約します。

- 個人が不足や問題を認識し、解決策を構築。
- コミュニティが問題をより広く議論し始めると、共有された解決策が出現します。
    - これらの解決策は、時には独占的であり、時にはオープンソースです。
- 解決策が成熟し、広範な合意に達すると、標準化が行われます。
- ステップ1に戻る。

これはコンポーネントについても同様に見られます。HTMLの初期の日々には、再利用可能なコンポーネントを作成する方法がありませんでした。応答して、私たち多くの人がクラスやコンポーネントを構築するための独自のライブラリを作成しました。それから最初の世代の共有ライブラリが登場しました：jQuery、MooTools、YUIなど。次に第二世代：AngularJS、Backbone、Knockoutなど。そして第三世代：Angular、React、Vueです。この時点で、共通のパターンがよく確立されていました。

さまざまな人々がこの第三世代のライブラリに忙しく取り組んでいる間、他の人々はすでに標準化の重要な作業を始めていました。
2013年5月、GoogleのVPであるLinus Upsonは、Googleがすでに数年間取り組んでいた技術であるGoogleの年次I/O基調講演中にウェブコンポーネントを発表しました。
非標準のライブラリの三世代が注目を集めようとしていたため、Googleの Web Components の提案をめぐる合意を得るには時間がかかりました。
ブラウザ間で実装されるまでさらに時間がかかりました。
2020年1月、新しいChromiumベースのEdgeブラウザがリリースされたこと。
すべてのブラウザが最終的に「Web Components v1」と呼ばれる機能をリリース。

## 2010
Model View ViewModel Controller(MVVC) というパターンが提案されました。
これは、Model View Controller(MVC) というパターンを拡張したもので、ViewModel という新しい概念を導入しました。
ViewModel は、View と Model の間のデータバインディングを担当し、View の状態を管理します。
これにより、View と Model の間の結合が緩和され、View と Model を独立してテストできるようになりました。
- knockout.js / https://github.com/knockout/knockout/releases/tag/v1.0.0
- AngularJS / https://github.com/angular/angular.js/releases/tag/v0.9.0
- Backbone.js
## 2011
Web platform における Model Driven Views というアイデアが提案されました。
https://fronteers.nl/congres/2011/sessions/web-components-and-model-driven-views-alex-russell

TraceUR compiler が classes を使った custom elements をサポートしました。
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes

## 2013
Google のチームはドラフトとして、Custom Elements の仕様を提案しました。
Custom Elements v0 として呼ばれていた。Shadow DOM に類似しているAPI仕様もありました。
https://web.archive.org/web/20130608123733/http://www.w3.org/TR/custom-elements/

Polymer プロジェクトが開始されました。
コンポーネントライブラリであり、MVCを使用して双方向のUIバインディングを提供しました。
仕様とともに、リリースされています。
https://web.archive.org/web/20130515211406/http://www.polymer-project.org/

同時期に、React がリリースされました。Polymerが提供する双方向のUIバインディング(MVCルート)とは異なり、React は単方向のデータフローを提供しました。
https://www.youtube.com/watch?v=GW0rj4sNH2w

## 2016, 2017
わずか1年前に、v0カスタムエレメント仕様を使用してPolymer 1.0がリリースされましたが、2016年にはカスタムエレメントv1仕様がリリースされました。
この新しいバージョンの仕様は後方互換性がなく、その結果、正常に機能するためには新しいバージョンの仕様への移行が必要となりました。v0実装を持たないブラウザのための一時的な解決策として、ポリフィルの使用は続けられました。
v1は2016年末にChromeに実装されていましたが、この仕様を起草したライブラリに再び採用されるのは、2017年にPolymer 2.0がリリースされるまでのことでした。
このため、YouTubeの新しいPolymerの書き直しが理論上ウェブコンポーネントの使用に向けて大きな一歩である一方で、問題を引き起こしました。v0実装を持たないFirefoxのようなブラウザは、ネイティブ実装より遅いポリフィルの使用を続けることを強いられました。

## 2018
2018年になって、Web Components は根を深くしてきました。
- Mozillaがv1仕様APIをFirefoxの安定版リリースに実装
    - 開発者はアプリ内でウェブコンポーネントのAPIをすべて、ブラウザ間で、そしてChrome以外のパフォーマンスに対する懸念なしに使用できるようになりました。
- Reactの一方向性はPolymerチームに受け入れられたようです。Polymerチームは、双方向バインディングから離れ、一方向バインドの LitElement への移行を発表しました。
- その後、LitElementは「Lit」と呼ばれる専用フレームワークになり、Polymerの後継として開発され、2019年にv1、2021年にv2がリリースされました。

フレームワークとしての比較
- https://coderpad.io/blog/development/web-components-101-framework-comparison/ 