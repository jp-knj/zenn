---
title: "デザイン: 認知負荷を軽減する"
---
## 概要
障害のある人々にとって、ユーザーインターフェイスデザインやビジュアルデザインをよりアクセスしやすくするための基本的な考慮点を紹介します。
これらのヒントは、Web Content Accessibility Guidelines (WCAG)の要件を満たすための良い実践となります。
関連する WCAG の要件、"Understanding"文書の詳細な背景、チュートリアルからのガイダンス、ユーザーストーリーなどへのリンクをたどってください。

### 十分なコントラストを提供する
前景のテキストは、背景色との間に十分なコントラストを持つ必要があります。これには、画像の上のテキスト、背景のグラデーション、ボタン、その他の要素のテキストが含まれます。これは、ロゴや写真に偶然表示されるテキストなどの偶発的なテキストには適用されません。以下のリンクでは、WCAG による最小コントラスト比とコントラストを確認する方法に関する詳細な情報を提供しています。 "Contrast ratio"は、より技術的に正確な用語 "輝度コントラスト比" の短縮形です。

| 不十分なコントラスト比 | 十分なコントラスト比 |
|-------------|------------|
|             |            |

https://www.w3.org/WAI/WCAG21/quickref/#contrast-minimum
https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum

### 色だけで情報を伝えない
色は情報を伝えるための便利な方法であるが、色を情報を伝える唯一の方法として使用すべきではない。
要素を区別するために色を使用する場合、色の知覚に依存しない追加の識別方法も提供すること。
例えば、必須のフォームフィールドを示すために色に加えてアスタリスクを使用するか、グラフの領域を区別するためのラベルを使用する。

| |  |
|-------------|------------|
|             |            |


| |  |
|-------------|------------|
|             |            |

https://www.w3.org/WAI/WCAG21/quickref/#use-of-color
https://www.w3.org/WAI/WCAG21/Understanding/use-of-color

https://www.w3.org/WAI/WCAG21/quickref/#consistent-identification
https://www.w3.org/WAI/WCAG21/Understanding/consistent-identification
### インタラクティブな要素を簡単に識別できるようにする
リンクやボタンなどのインタラクティブな要素を簡単に識別できるように、独自のスタイルを提供すること。例えば、マウスのホバーやキーボードのフォーカス、タッチスクリーンのアクティベーション時にリンクの外観を変更します。インタラクティブな要素のスタイルや命名が、ウェブサイト全体で一貫して使用されていることを確認します。

#### 異なるリンクの状態のためのユニークなスタイル
- リンクをテキストから目立たせるスタイル
  一部の人々はマウスを使えず、ウェブページをナビゲートするためにキーボードだけを使用します。
- マウスホバースタイル / ナビゲートするためのキーボード
- キーボードフォーカススタイル / ナビゲートするためのキーボード
- タッチまたはクリックのスタイル / ナビゲートするためのキーボード

https://www.w3.org/WAI/WCAG21/quickref/#focus-visible
https://www.w3.org/WAI/WCAG21/Understanding/focus-visible

### 明確で一貫性のあるナビゲーションオプションを提供する
ウェブサイト内のページ間のナビゲーションには、一貫性のある命名、スタイリング、配置が必要です。ウェブサイトのナビゲーションには、サイトの検索やサイトマップなど、複数の方法を提供します。ユーザーがウェブサイトやページのどこにいるのかを理解するのを助けるために、方向付けの手がかり、例えば、パンくずリストや明確な見出しを提供します。

https://www.w3.org/WAI/WCAG21/quickref/#consistent-navigation
https://www.w3.org/WAI/WCAG21/Understanding/consistent-identification

https://www.w3.org/WAI/WCAG21/quickref/#multiple-ways
https://www.w3.org/WAI/WCAG21/Understanding/multiple-ways

### 認知的な困難を持つ人に一貫性とナビゲーションが役立つ方法
フォーム要素には明確に関連付けられたラベルが含まれていることを確認する。
すべてのフィールドには、フィールドの隣に記述的なラベルが付いていることを確認してください。左から右の言語では、ラベルは通常、フィールドの左または上に配置されますが、チェックボックスやラジオボタンでは、通常右に配置されます。ラベルとフィールドの間に余分なスペースがあるのを避けてください。

#### ラベルと入力フィールドの関連付け
https://www.w3.org/WAI/WCAG21/quickref/#labels-or-instructions
https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions

https://www.w3.org/WAI/WCAG21/quickref/#headings-and-labels
https://www.w3.org/WAI/WCAG21/Understanding/headings-and-labels

### 明確なラベル付けが認知的な困難を持つ人を助ける方法
簡単に識別できるフィードバックを提供する。
フォームの送信の確認、何かが間違っているときのユーザーへの警告、ページの変更の通知など、インタラクションのフィードバックを提供します。指示は簡単に識別できるようにする必要があります。ユーザーのアクションが必要な重要なフィードバックは、目立つスタイルで提示する必要があります。

#### エラー、アイコン、背景色を使用してエラーを目立たせる
https://www.w3.org/WAI/WCAG21/quickref/#error-identification
https://www.w3.org/WAI/WCAG21/Understanding/error-identification

https://www.w3.org/WAI/WCAG21/quickref/#error-suggestion
https://www.w3.org/WAI/WCAG21/Understanding/error-suggestion

### 関連するコンテンツをグループ化するために見出しとスペーシングを使用する
コンテンツ間の関係を明らかにするために、ホワイトスペースと近接性を使用します。見出しをスタイル化してコンテンツをグループ化し、 clutter を減少させ、スキャンや理解を容易にします。

### 異なるビューポートサイズのデザインを作成する
ページ情報が、モバイル電話やズームされたブラウザウィンドウなどの異なるサイズのビューポートでどのように表示されるかを検討してください。ヘッダーやナビゲーションなどの主要な要素の位置や表示を変更して、スペースを最適に使用します。読みやすさと可読性を最大化するために、テキストのサイズと行幅を設定します。

#### コンテンツとナビゲーションが、小さいモバイルスクリーンに適応する
広いウィンドウと小さいテキストの表示では、主要なコンテンツのために複数の列が使用され、ナビゲーションオプションや 2 次情報が表示されます。
狭いウィンドウや大きなテキストの表示では、主要なコンテンツは単一の列で表示されます。ナビゲーションオプションはアイコンで示され、2 次情報もアイコンで表示されるようになっています。

https://www.w3.org/TR/mobile-accessibility-mapping/#h-small-screen-size
https://www.w3.org/TR/mobile-accessibility-mapping/#mobile-accessibility-considerations-related-primarily-to-principle-3-understandable

### デザインに画像やメディアの代替を含める
画像やメディアの代替をデザインに組み込むための場所を提供します。例えば、以下が必要となる場合があります。

- オーディオのトランスクリプトへの可視リンク
- ビデオの音声説明版への可視リンク
- アイコンやグラフィカルボタンと共に表示されるテキスト
- テーブルや複雑なグラフのキャプションや説明
- 非テキストコンテンツの代替を提供するために、コンテンツ制作者や開発者と連携

https://www.w3.org/WAI/WCAG21/quickref/#non-text-content
https://www.w3.org/WAI/WCAG21/Understanding/non-text-content

### 自動的に開始するコンテンツのコントロールを提供する
ユーザーがアニメーションや自動再生音を停止できるように、可視のコントロールを提供します。これには、カルーセル、画像スライダー、背景音、ビデオが含まれます。

#### カルーセルデザインに再生と停止および選択コントロールを表示

https://www.w3.org/WAI/WCAG21/quickref/#audio-control
https://www.w3.org/WAI/WCAG21/quickref/#pause-stop-hide