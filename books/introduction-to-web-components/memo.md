# 実際に Web Components が使われている現場について
- YouTube

# Web Components の歴史

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
