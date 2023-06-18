---
title: "デザインシステムを定義しよう - ファンデーションのアクセシビリティ、カラーについて"
---
## ファンデーション 

https://youtu.be/o9CloFonY0E

ファンデーションは、デザインシステムの根幹です。ビジュアルスタイル、カラー、タイポグラフィ、コンポーネントなどのプロダクト体験や、構造、動作、規約などのパターンを作成するために使用する構成要素です。

![](https://storage.googleapis.com/zenn-user-upload/b83efb94d064-20230604.png)

ファンデーションに含めることができるものは沢山ありますが、それはプロダクトによって異なります。ここでは考慮すべきいくつかの重要なポイントについて説明します。

### アクセシビリティ
ファンデーションに含めるべきプラクティスのひとつに、アクセシビリティがあります。アクセシビリティは、a11y という略称で呼ばれることもありますが、障害のある人に焦点を当てたインクルーシブデザインの一つです。
ハンディキャップには私たちの周りの世界の見え方、聞こえ方、話し方、触れ方、理解の仕方に影響を与えることがあります。ハンディキャップには、永久的なもの（失明など）、一時的なもの（骨折や失声など）、状況的なもの（運転中の視界の悪さなど）などがあります。

![](https://storage.googleapis.com/zenn-user-upload/bdaaac9b76fb-20230604.png)

ほとんどの人は、一時的か永続的に、人生のどこかでハンディキャップを経験したことがあるのではないでしょうか
デザイナーとして、私たちの決断はそのようなユーザーに影響を与えてしまうかもしれません。もし、私たちが作ったものにユーザがアクセスできなかったり、使えなかったりしたらプロダクトを通してユーザに重要でない、忘れられた、歓迎されていないことを感じさせる可能性があるのです。

![](https://storage.googleapis.com/zenn-user-upload/b4bc9e9d356c-20230618.png)

アクセシビリティは、多くのデザインシステムで基礎となっていることがわかります。アクセシビリティは、よりアクセシブルな体験を大規模に創造するために、チームを強化する効果的な方法です。

もっと詳しく知りたい方は、以下のリンク先のリソースをチェックしてみてください

https://www.youtube.com/watch?v=1wyCtbZEViE&t=508s
https://www.thebookonaccessibility.com/
https://www.magentaa11y.com/web/
https://colorblindaccessibilitymanifesto.com/
https://adhoc.team/playbook-accessibility/
https://umusic.co.uk/Creative-Differences-Handbook.pdf
https://www.getstark.co/library/
https://abookapart.com/products/accessibility-for-everyone
https://makeitfable.com/

## カラー
カラーは個性を表現し、感情を呼び起こし、行動に影響を与えることができます。文化によって同じ色でも異なる意味を持つこともあります。

![](https://storage.googleapis.com/zenn-user-upload/719e0e319748-20230604.png)

効果的なカラーパレットは、色の選択肢を提供してくれるます。

![](https://storage.googleapis.com/zenn-user-upload/f5e088717723-20230618.png)

多すぎると混沌としてしまいますが、少なすぎるとデザインの自由度が低くなってしまいます。特に、異なるテーマに対して色をどのように適応させるか考えるのは難しい問題です。
例えば、プロダクトのダークモードを作る場合、明るい色と暗い色を入れ替えるだけでは不十分です。色の濃淡のバリエーションが必要になるかもしれません。
また、複数のプロダクトを展開する企業では、それぞれのプロダクトの個性を魅せたい思いがあると思うのですが、そのために色を増やしすぎると、使い方がバラバラになったり、ブランドが崩れてしまったり、カラーパレットが使いづらくなったりします。

バランスの取れたカラーパレットを作るには、システムに必要な機能と色を照らし合わせることが重要です。メッセージ、ステータス、優先度、アクションなどの機能に対して、1つの色の値が意味を持つようになります。

![](https://storage.googleapis.com/zenn-user-upload/952a342b3ae0-20230604.png)

## カラーとアクセシビリティ
カラーには、アクセシビリティに関する考慮事項があります。[カラーのコントラスト](https://www.w3.org/WAI/WCAG21/quickref/#audio-control)は、要素をはっきりと見る視力に影響します。
コントラストとアクセシビリティを測定するための[ツール](https://www.figma.com/community/search?resource_type=plugins&sort_by=relevancy&query=color+contrast&editor_type=all&price=all&creators=all)はたくさんあります。Figmaでは、デザインが [Web Content Accessibility Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)に適合しているかどうかを確認するためのプラグインを見つけることができます。

アクセシビリティ・レベルには、A、AA（ダブルA）、AAA（トリプルA）の3つがあります。Aの数が多いほど、高い水準となります。このガイドラインは最低限なので、AA以上を目指しましょう。アクセシビリティに重要なのは、コントラストだけではありません。誰もが同じ程度に色を見たり区別したりできるわけではありません。色だけで状態を伝えたり、要素に意味を持たせたりすることはできないのです。

[色だけでなく、シンボルやアイコン、ヘルパーテキスト](https://www.w3.org/WAI/WCAG21/quickref/#non-text-content)を追加して、要素をよりよく区別できるようにすることを検討してください。色に関する詳しいガイダンスについては、[Schema 2021の Asana の Design Systems Teamのトークにある Design tokens](https://www.youtube.com/watch?v=ylDed18OVdY) をご覧ください。

![](https://storage.googleapis.com/zenn-user-upload/4b8f1a6b6367-20230604.png)

### ユーザーとテストする
導入したカラーパレットやアクセシビリティは完璧ではありませんし、時には予想外の結果が出ることもあります。デザインをテストし、実際のユーザーからフィードバックを得ることに代わるものはありません。特に、支援技術[^1]を使用しているユーザーなど、アクセシビリティのニーズを持つ人を含めるようにしてください。

[^1]: 障害者等が利用している装置やソフトウェアのこと 
https://www.aao.ne.jp/column/links/1-3.html