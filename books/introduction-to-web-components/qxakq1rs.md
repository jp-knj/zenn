---
title: "Web Components の全体像"
---
## プラットフォーム相互運用性

Webコンポーネントの最も重要な側面の一つは、コンポーネントとプラットフォーム間の相互運用性をどのように可能にするかです。この分野の現在および将来の機能をいくつか見てみましょう。

### カスタムエレメント

自律カスタムエレメント（完全サポート） - Webコンポーネントv1のこのコア機能により、customElementsグローバルにクラスを登録することで、HTMLElementから継承するカスタムエレメントを定義できます。基本的なライフサイクルコールバックと観測された属性も仕様の一部です。
カスタマイズ可能な組み込みカスタムエレメント（却下） - 元々はHTMLParagraphElementのような組み込み要素から継承することを可能にする提案がありましたが、WebKitの実装者がいくつかの技術的な問題を発見したため、この仕様は却下されました。将来的には削除される可能性が高く、避けるべきです。より良い代替手段として「カスタム属性」を参照してください。

### エレメントインターナルズ

v1以降の新しいAPIであるElementInternalsは、カスタムエレメントを既存のDOMサブシステムとより深くプラットフォームレベルで統合することを可能にします。

シャドウルートアクセス（完全サポート） - このシンプルな追加により、コンポーネントの作成者は、クローズドモードエレメントのシャドウルートインスタンスを取得できます。これがなければ、クローズドモードの宣言的シャドウDOMを持つ要素がランタイムでルートにアクセスする方法はありません。
フォーム関連カスタムエレメント（完全サポート） - この新しい一連の機能により、カスタムエレメントはフォームの検証、送信、リセットに完全に参加できるようになります。
デフォルトのアクセシビリティロール、状態、およびプロパティ（ほとんどサポート） - この新しいAPIセットにより、カスタムエレメントのデフォルトのアクセシビリティ特性を、ホスト要素の外部向け属性を介さずに直接プラットフォームと内部的に通信することで設定できます。これにより、消費者によって誤って削除される可能性があります。現時点では、新しいAPIはFirefoxを除くすべての主要なブラウザでサポートされており、Firefox用のポリフィルがあります。FirefoxがすでにElementInternals APIの残りの部分を実装しているので、近い将来にこれを出荷しないと驚くでしょう。
合成選択（コンセンサス/仕様なし）

この改善は、選択オブジェクトに対する新しいgetComposedRange() APIを提案しており、範囲の開始と終了が複数のシャドウルートを越えることを可能にします。また、ブラウザがこれらのシナリオをどの

ように処理するかという一貫性も向上させます。ストローマンAPI提案には一般的なコンセンサスがありますが、ブラウザが実装を進める前に完全な仕様がまだ必要です。これは、Webコンポーネント開発の通常のコース中に遭遇する可能性があるものではありません。主にリッチテキストエディタの実装に関連しています。

### カスタム属性（識別）

この機能は必ずしもWebコンポーネントの一部ではありませんが、Webコンポーネントが提供することを目的としたシナリオのセットと高い重複があります。ストローマンは、Webコンポーネントと同様のパターンに従って、任意のHTML要素にアタッチできる再利用可能な動作の作成を可能にすることを提案しています。たとえば、任意のHTML要素に適用したいMaterial Designのリップルエフェクトを想像してみてください。これができたらいいと思いませんか？

```html
<button material-ripple>Click Me</button>
```

TPAC 2022で準備したストローマン提案では、これのプログラミングモデルがどのように見えるかを示しています：

```javascript
class MaterialRipple extends Attr {
  // ownerElement inherited from Attr
  // name inherited from Attr
  // value inherited from Attr
  // ...

  connectedCallback () {
    // called when the ownerElement is connected to the DOM
    // or when the attribute is added to an already connected owner
  }

  disconnectedCallback () {
    // called when the ownerElement is disconnected from the DOM
    // or when the attribute is removed from a connected owner
  }

  attributeChangedCallback() {
    // called when the value property of this attribute changes
  }
}

customAttributes.define("material-ripple", MaterialRipple);
```

Webコンポーネントと同じパターンとライフサイクルに気付くでしょう。これは、却下されたカスタマイズ可能な組み込みカスタムエレメント提案の一部であったis属性よりも優れた、より堅牢な代替手段を提供します。