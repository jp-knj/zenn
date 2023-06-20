---
title: "デザインシステムを育もう - ドキュメントとシステムを保守する"
---

https://youtu.be/_sTWtnNU1L0

デザインシステムの利用者の心をつかむには、効果的なドキュメントは必須です。

### ドキュメントとは？
Kai と Habitz チームは、いくつかの命名規則を定義し、スタイルとコンポーネントの説明を書きました。しかし、ドキュメンテーションはこれだけにとどまらないことがあります。

たとえば、次のようなことがあります。

- パターンの正しい使い方と間違った使い方の例を記載する。
  ![](https://storage.googleapis.com/zenn-user-upload/19b986583fcf-20230614.png)

- コンポーネントのアップデートを追跡するために変更ログを使用する
  ![](https://storage.googleapis.com/zenn-user-upload/2b0e6e2e487a-20230614.png)

- コード例や開発者の実装へのリンクを追加する
  ![](https://storage.googleapis.com/zenn-user-upload/eeb6fc2070ca-20230614.png)

- 効果的なドキュメントは、利用者のニーズを予測し、利用者に直接語りかけるような方法で表示されます。例えば、開発者は Token と Properties を相互参照する必要があるかもしれませんし、ライターにはエラーメッセージを作成するためのガイダンスが必要かもしれません。
  ![](https://storage.googleapis.com/zenn-user-upload/94577bcae74f-20230614.png)

システムのどの部分に捕捉事項が必要かを考えてみてください。異なるオーディエンスを知ることで、どんなメリットがあるのでしょうか？

![](https://storage.googleapis.com/zenn-user-upload/7cbb1ad8eae2-20230614.png)

### ドキュメントはどこに置くべき？
ドキュメントをどこに置くか、いかに見つけやすくするかは、デザインシステムの効果に影響を与える可能性もあります。デザインシステムが専用のウェブサイトで公開されているのをよく見かけます、

![](https://storage.googleapis.com/zenn-user-upload/649b41cd7cd4-20230614.png)

それよりも、Storybook や Notion のようなツールを使ったり、デザインシステムと同じ場所にドキュメントを置いたりした方が、早く立ち上げることができるかもしれません。

#### ドキュメントにリンクを追加する
デザインシステムが Figma にあり、そのドキュメントが別の場所にある場合、どのコンポーネントやコンポーネント・セットからもドキュメントにリンクして、簡単にアクセスすることができます。

https://help.figma.com/hc/en-us/articles/7938814091287

#### Habitz
Habitz のチームは小規模なため、既存のデザインファイルを使用して、資産とデザインドキュメントの両方をホストすることにしました。これは彼らの現在のワークフローに合っており、すべてを一箇所にまとめておくことができます。

このファイルには、次のようなページが用意されています：
- Foundations ：スペーシング、タイポグラフィ、カラーなど；
- Components ：ボタン、カード、トグルなど；
- Patterns（デイセレクター、ナビゲーション、リストなど）。

![](https://storage.googleapis.com/zenn-user-upload/0f13bee43d2b-20230614.png)

各アセットのコレクションは、ラベルと、追加のメモやコンテキストとともにフレームに配置されます。

### Figmaでのドキュメンテーション
Habitz のチームは、8pxグリッドシステムの修正版を使用しています。彼らは、スタイルやコンポーネントの外側に、その情報を明示的に伝えたいと考えています。

![](https://storage.googleapis.com/zenn-user-upload/904d88051173-20230614.png)

彼らは、共通の スペース値を取得するためのいくつかの要素を作成しました。これにより、デザイナーはどの値を使うべきかを知ることができ、デザインと実装の整合性をより高めることができます。

![](https://storage.googleapis.com/zenn-user-upload/26d0b6c399c7-20230615.png)

これらの スペース定義がコンポーネントでないことにお気づきでしょうか。それは、これらのアセットがデザインに使用されることを想定していないためです。

![](https://storage.googleapis.com/zenn-user-upload/97a3776a89e1-20230615.png)

むしろ、何もないスペースをイメージしてもらうためのイラストであり、システムの決定事項を記録し、利用者に参照させるために存在するものなのです。

![](https://storage.googleapis.com/zenn-user-upload/6ee2960a339c-20230615.gif)

スペースシステムは、レイアウトグリッド、padding、そしてこのドキュメントにあるように、Figma のナッジ値など、他のものの基礎も設定します。

![](https://storage.googleapis.com/zenn-user-upload/45da4f47ce0a-20230615.png)

### Design Linter の活用
Design Linter は、間違ったスペースやスタイルの欠落など、デザインシステムの要件を満たしていない部分について、デザインを監査します。Daniel Destefanis の Design Lint や Roller by Contrast のような Linter をFigmaコミュニティでチェックして、自分のシステムで使えるかどうか確認してみてください。

https://www.figma.com/community/plugin/801195587640428208/Design-Lint

https://www.figma.com/community/plugin/751892393146479981/Roller-%C2%B7-Design-Linter
