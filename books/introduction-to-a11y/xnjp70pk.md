---
title: "アクセシビリティを評価する手法: WCAG-EM"
---
前回のセクションで WCAG について、知ることができました。今回では、WCAG を活用してアクセシビリティ向上に取り組むときに、Web アクセシビリティの評価手順について解説していきます。
この記事は、Web アクセシビリティ評価のステップを簡潔にまとめたものです。エンジニア、デザイナー、PdM の皆様に、Web アクセシビリティの評価や改善をするときの、参考として役立つことを願っています。

## WCAG-EMの存在とその目的
WCAG-EM（Website Accessibility Conformance Evaluation Methodology）は、Web アクセシビリティの評価と向上を目的としたガイドラインです[^1]。この記事で、そのガイドラインと方法論を詳しく解説します。

※ [Website Accessibility Conformance Evaluation Methodology](https://www.w3.org/TR/WCAG-EM/) を参考にしており、以下の内容は筆者の意訳になります。正確な情報を知りたい方は、前述のリンクを参考にしてください。
※ ドキュメントに抵抗がある方は、WAIC（情報通信アクセス協議会ウェブアクセシビリティ基盤委員会）の [JIS X 8341-3:2016 試験実施ガイドライン](https://waic.jp/docs/jis2016/test-guidelines/202012/) を参考にしてください。

## WCAG-EMの基本
### WCAG-EM とは 
**WCAG-EM**は、Web コンテンツのアクセシビリティを評価するための手順を示す方法論です。これは、WCAG に基づいており、アクセシビリティガイドラインを深く理解した上で適用することが推奨されています。
Web アクセシビリティの評価と向上を目指すための基本的な概念を提供するガイドラインです。特に大規模な Web コンテンツのアクセシビリティ向上を前提として策定されていますが、コンポーネント単位やページ単位での改善にも効果的です。

## WCAG-EM の評価ステップ
WCAG-EM の評価は、以下の 5 つの主要なステップから成り立っています。

![](/images/books/a11y/wcag-em.png)
※ WCAG-EM の評価ステップは、 各ステップを行き来することになります。コンテンツの調査や選定をした後、スコープの定義に戻ることもあります。

1. **スコープの定義**：評価の対象となる Web コンテンツ、評価の目標、および WCAG の適合レベルである A、AA や AAA を決定します
2. **コンテンツの調査**：Web サイト内の主要なページや頻繁に使用されるページ、さまざまなコンテンツの種類を特定します
3. **コンテンツの選定**：すべてのページを評価するのは難しいため、代表的なページやランダムに選ばれたページを評価の対象とします
4. **コンテンツの評価**：選択されたページが WCAG の基準に準拠しているかを確認します
5. **評価結果の報告**：評価の結果をまとめ、サイトのアクセシビリティレベルを示す結果の報告書を作成します

ユーザーにとって使いやすい Web サイトを提供するための鍵となります。WCAG-EM を適切に活用することで、その目標に一歩近づくことができます。

## スコープの定義
全体的な評価対象を定義する基本的なステップです。このステップは、評価手順の後続のステップに大きく影響します。

### 適合目標の定義
WCAG 2.0 の適合レベルである A、AA や AAA を評価の目標として定義します。一般的には、WCAG 2.0 レベル AA が推奨されています。

適合レベルを超えて、評価することは有益です。Web コンテンツが特定の適合レベルを満たしていても、より高い基準で評価します。改善のための領域や潜在的な拡張に関する洞察を得ることができます。適合目標を超えて評価することで、組織は自分たちがどこに立っているのか、将来どのようなステップを踏む必要があるのかを明確にできます。この前向きなアプローチは、アクセシビリティの継続的な改善を保証しやすくなります。

### アクセシビリティサポートの基準を定義
アクセシビリティサポート[^2] は、Web 技術が実際に障害を持つ人々にとって機能するかどうかを判断するためのものです。そのための基準として、動作することが期待される OS、ブラウザ、AS(Assistive Technology)[^3]、やユーザーエージェントの組み合わせを策定します。

例えば、Web 技術が理論的にはアクセシビリティの要件を満たしているとしても、実際の AS 適切に動作しないとき、「アクセシビリティサポートされていない」と見なされる可能性があります。

**アクセシビリティサポートの基準の具体例**
| OS      | ブラウザ    | AS        |
|---------|---------|-----------|
| Windows | Chrome  | NVDA      |
| Windows | Firefox | JAWS      |
| macOS   | Safari  | VoiceOver |
| Android | Chrome  | TalkBack  |
| iOS     | Safari  | VoiceOver |

## Web コンテンツの調査
このステップでは、Web コンテンツを調査します。その使用方法、目的、機能についての理解を深めます。特に開発チームの外部にとっては、Web サイトの全体像を把握するのは難しいでしょう。評価者が対象の Web サイトについてさらに学ぶことで、後のステップで洗練されます。

- Web サイトの全ての部分にアクセスできる必要があります。特定のエリアにアクセス制限がある場合は、権限は前もってもらっておきましょう。
- 評価の初期段階で、Web ページの基本的なアクセシビリティをチェックします。この段階でのチェックは、後の詳細な評価のための基盤を築く役割を果たします。
- 色のコントラスト、文書構造やナビゲーションの一貫性など、問題点や改善点をメモしておきます。

### Webサイトの構造と内容の特定
#### 主要なページやセクションを特定
これにはホームページや、ヘッダー、ナビゲーション、フッターからアクセスできる主要なページが含まれます。Web サイトの構造を理解するために、サイトマップを作成することをおすすめします。

#### 主要な機能を特定
これには、商品の購入機能やユーザー登録機能のような主要な機能が含まれます。全ての機能を網羅する必要はなく、Web サイトの主要な目的や目標に関連する機能を中心に特定します。

##### 事例
- オンラインショッピング : いろいろな商品やその価格を見て、気に入ったものを選びます。次に、その商品をカートに入れ、配送先や支払い方法を決めて注文します
- アカウント作成 : 新しいサイトでアカウントを作成する際、まず「私はロボットではありません」というテストをパスする必要があります。このテストをクリアした後、ユーザー名やパスワードなどの必要な情報を入力する登録フォームに進むことができます。最後に、入力した情報を確認して、アカウントの登録を完了します

#### Web ページの種類やスタイルの特定
異なるデザインや機能を持つページは、アクセシビリティのサポートが異なる可能性があるため、これらのページを特定することが重要です。

##### 事例
**ニュースサイトの記事ページ**
- 特徴： 多くのテキスト、画像、動画、関連記事へのリンクなどが含まれる 
- アクセシビリティの課題： 視覚障害のあるユーザーが画像の内容を理解するための代替テキストの提供、動画の字幕や音声解説の提供、テキストのコントラストの確保など

**E コマースサイトの商品ページ**
- 特徴： 商品の画像、詳細説明、価格、レビュー、カートに追加するボタンなどが含まれる
- アクセシビリティの課題： カートに追加するボタンの大きさや位置の最適化、商品の詳細説明の読みやすさ、レビューのフォントサイズの調整や色のコントラストの確保など 

**SNS のプロフィールページ**
- 特徴： ユーザーの写真、自己紹介文、フォロワー数、投稿一覧などが表示される
- アクセシビリティの課題： ユーザーの写真に対する代替テキストの提供、リンクやボタンの明確な識別、投稿の文字サイズや背景色の調整など

### Web サイトが依存している技術の特定
HTML, CSS, JavaScript（Client-side Scripting, Server-side Scripting）や WAI-ARIA、SVG などの技術が評価の対象となります。

以上のステップを通じて、Web サイトのアクセシビリティを効果的に評価するための準備を整えることができました。

## 評価対象コンテンツの選定
このステップでは、評価対象の Web サイトを代表する Web ページや Web ページの状態のサンプルを選択します。この選択の目的は、評価結果が Web サイトのアクセシビリティの実際のパフォーマンスを適切に反映することを確認するためです。

### 構造化されたコンテンツを含める
Web サイトの各部分を代表する Web ページや Web ページの状態を選択します。これには、Web サイトの共通ページ、主要な機能、ページの種類、使用されている Web 技術などが含まれます。

サンプルの選択は、Web サイトのサイズ、複雑さ、一貫性、開発プロセスの遵守など、多くの要因に依存します。このステップを通じて、Web サイトのアクセシビリティの評価を効果的に行うためのコンテンツを選択できます。

## コンテンツの評価
このステップでは、選定された Web コンテンツとその状態を詳細に評価します。そして、構造化されたサンプルとランダムに選択されたサンプルを比較します。この評価では定義された目標の適合性レベルでの WCAG 2.0 の 5 つの適合要件に従って行われます。

### WCAG 2.0 の 5 つの適合要件
Web アクセシビリティは、すべての人々が Web コンテンツをアクセスし、利用することを可能にするための重要な概念です。WCAG 2.0 は、Web コンテンツのアクセシビリティを評価するための基準を提供しています。このセクションでは、WCAG 2.0 への適合性に関する要件とその理解について説明します。

### 適合要件
Web ページが WCAG 2.0 に適合するためには、以下のすべての要件を満たす必要があります。

**適合レベル**: 以下の適合レベルのいずれかが完全に満たされていること。
- **レベルA**: 最低限の適合。Web ページがすべてのレベル A の成功基準を満たすこと（例： 画像に代替テキストを提供する）
- **レベルAA**: Web ページがレベル A とレベル AA の成功基準の両方を満たすこと（例： 色のコントラストを確保する）
- **レベルAAA**: Web ページがレベル A、レベル AA、およびレベル AAA の成功基準のすべてを満たすか（例： 動画に字幕や音声解説を提供する）

#### Web ページ単位の確認
選定されたコンテンツの各 Web ページは、WCAG 2.0 の適合要件に基づいて確認されます。ですが、特定の操作やデータ入力はありません。Web ページにある全てのコンポーネントがこの確認の対象となります。ページ全体の基準を満たしているかどうかに基づいて評価されます。部分的な基準を満たしているかどうかは評価されません。

**事例**
オンラインショッピングには、商品の一覧ページがあります。商品の画像、価格、説明などの情報が表示されています。各ページに存在するすべてのコンポーネント（例： ボタン、リンク、画像、テキストなど）が確認の対象となります。例えば、商品の一覧にある画像の代替テキストが一部欠けているとき、そのページ全体が適合レベルを満たしているとはいえません。

#### シナリオ単位の確認
ある Web ページが特定のタスクを達成するためのシナリオ（一連のステップ）として機能するとき、そのシナリオ全体を形成するすべての Web ページは、定義された適合レベルを満たしている必要があります。

**事例**
オンラインショッピングには、製品の選択から購入までの一連のページがあります。シナリオの開始から終了までのコンテンツが適合している必要があります。

- 商品の選択 : ここでユーザーは商品を閲覧し、希望の商品をカートに追加します。このページは、商品の画像、価格、説明などの情報を明確に表示する必要があります
- カートへ追加 : ユーザーはここで選択した商品の一覧を確認し、数量や色などのオプションを変更できます。また、不要な商品をカートから削除できます
- 支払い情報の入力 : このページでは、ユーザーは支払い方法を選択し、必要な情報（クレジットカード情報、住所など）を入力します
- 購入の確認 : ユーザーは注文の詳細を確認し、購入を確定します

これらの各コンテンツは、アクセシビリティの基準に従って設計され、実装されている必要があります。アクセシビリティ基準を満たしていないと、ユーザーはタスクを完了できない可能性があります。そのため、このシナリオに必要なコンテンツは定義された適合レベルを満たしている必要があります。

## 評価結果の報告
評価の詳細を正確に記録することは、あとで参照したり、問題が発生したときの対応のために役立ちます。評価者や依頼者など、関わるすべての人々にとって、適切な取り扱いと共有が求められます。

記録すべき内容は以下になります。
- 評価対象の Web ページやその状態のアーカイブ
- 使用した評価ツールやブラウザ、支援技術などの情報
- 評価中の Web ページのスクリーンショット
- 適合性を評価する方法や手順

### 評価結果を社外に報告
評価結果を社外に発表するときは、以下の情報を含めることが推奨されています。
- 発行日
- 使用したガイドラインの情報（タイトル、バージョン、URI）
- 目指したアクセシビリティの適合レベル（A、AA、AAA）
- 評価対象の Web サイトの詳細 
- 使用した Web 技術の情報

:::details アクセシビリティのテストに役立つツール。
#### デザイン
- **WebAim コントラストチェッカー** : テキストとグラフィックスの色のコントラストをチェック

https://webaim.org/resources/contrastchecker/

- **Color Oracle** : 色覚異常シミュレータ

https://colororacle.org/

- **Stark** : Figma のアクセシビリティプラグイン

https://www.figma.com/community/plugin/732603254453395948/stark-accessibility-tools

- **The A11y Project** : デザインと UX に関してのリスト
  
https://www.a11yproject.com/resources/#design-and-user-experience

- **Accessible Prototypes Playground** : スクリーンリーダーに特化したアクセシブルな Figma プロトタイプ

https://www.figma.com/community/file/1167124335986833540/accessible-prototypes-playground

#### コーディング 

- **Eslint-plugin-jsx-a11y lintingプラグイン**: React アプリでのアクセシビリティ問題を見つける

https://www.npmjs.com/package/eslint-plugin-jsx-a11y

- **Cypress axe**: axe-core でのアクセシビリティチェックを実行する Cypress コマンド

https://github.com/component-driven/cypress-axe

- **@axe-core/playwright**
https://playwright.dev/docs/accessibility-testing

- **WebAimのキーボードアクセシビリティガイド**: すべての対話型要素がキーボードアクセス可能であり、フォーカスフィードバックが表示されていることをチェック

https://webaim.org/techniques/keyboard/

- **ChromeLens devツール拡張**: 'trace tab path'を使いタブ順序の論理性をチェックします。また、Chrome や Firefox では'show tabbing order'を活用します。

https://chromelens.github.io/chromelens/

- **Tiny Helpers**: Web 開発者のための無料の単一目的オンラインツールのコレクション

https://tiny-helpers.dev/accessibility/

#### スクリーンリーダー

- **A11ycasts Voiceover**

https://www.youtube.com/watch?v=5R-6WvAihms

- **WebAimのVoiceoverガイドを読む**

https://webaim.org/articles/voiceover/

- **A11ycasts NVDA**

https://www.youtube.com/watch?v=Jao3s_CwdRU

- **WebAimのNVDAガイドを読む**

https://webaim.org/articles/nvda/

#### チェックリスト
ガイダンスのためのチェックリストとして使用してください。チェックリストはすべてのシナリオを網羅しているわけではありませんし、テストに置き換わるものでもありません。

- **A11y Project および WebAIM**: 平易な英語での包括的なチェックリスト。

https://www.a11yproject.com/checklist/

- **WebAIM**: 平易な英語での包括的なチェックリスト。

https://webaim.org/standards/wcag/checklist

- **WebAim webアクセシビリティ評価ガイド**: コンテンツタイプごとのアクセシビリティチェック。

https://webaim.org/articles/evaluationguide/

- **アクセシビリティチェックリスト**: 主要な考慮事項の Figma チェックリスト。

https://www.figma.com/proto/V7YQVtYIHS5gWKalbTvubq/%F0%9F%9F%A2-Handoff-Documentation?page-id=163%3A66379&node-id=163%3A66926&viewport=496%2C49%2C0.15&scaling=scale-down-width&hide-ui=1

[^1]:[Website Accessibility Conformance Evaluation Methodology](https://www.w3.org/TR/WCAG-EM/)
[^2]:アクセシビリティサポートは、Web 技術がアクセシビリティツール（例： スクリーンリーダーや音声認識ソフト）や AS（例： 点字ディスプレイや音声出力）によって適切に動作することを示します。
[^3]:障がいを持つ人々が情報やサービスにアクセスするのを助けるための技術やツールを指します。これには、スクリーンリーダーや拡大ソフトウェア、音声認識ソフトウェア、特別なキーボードやマウスなどが含まれます。web.dev にある [Assistive Technology](https://web.dev/learn/accessibility/test-assistive-technology) についての記事を参考にしてください。 
