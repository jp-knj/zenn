---
title: "デザインシステムを構築しよう - アイコンとイラストのスタイルを作成する"
---
## アイコンとイラストレーション
Habitz のチームは、大胆なネオブルータリズムのスタイルに気まぐれな要素を加えた、ブランドの個性をサポートするアイコンとイラストを使用しています。

![](https://storage.googleapis.com/zenn-user-upload/c7a19df07f49-20230605.png)

## アイコンコンポーネントの作成
3種類のアイコンサイズに対応することで足並みを揃えた 16, 24, 32 の3種類を用意しました。まず、真ん中のアイコンサイズから作り始める。
各アイコンを24×24の枠内に配置し、先に作成したアイコングリッドに合わせます。こうすることで、アイコンを入れ替えたときに、アイコンの中心が揃い、大きさも一定になります。16と32のアイコンについても、同様の作業を行いました。

![](https://storage.googleapis.com/zenn-user-upload/828571ed52f1-20230605.png)

https://help.figma.com/hc/en-us/articles/360039150413

コレクション内のすべてのアイコンを選択した状態で、ツールバーから複数のコンポーネントを作成し、それぞれのアイコンをコンポーネント化する。
Kai さんは、Habitz の開発チームと連携し、アイコンの命名が実装と一致すること、そしてデザインシステムを使っている人が検索しやすいことを確認しました。

彼らは、`icons ／size ／name` という命名構造に合意し、すべてのアイコンにそれを適用しました。これにより、アセットパネルでアイコンをグループ化するとき、まずアイコン、次にサイズという順番で表示されるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/6e491c77db76-20230605.png)

アイコンはコンポーネントであるため、用途を伝えるための説明を加えることができます。またアイコンの検索を容易にするために、検索キーワードを追加することもできます。

https://help.figma.com/hc/en-us/articles/7938814091287

## イラストレーション
Habitz ではイラストを使用することで、ブランドを強化し、プロダクトに華やかさを添えています。アイコンと同様に、イラストもフレームサイズを統一し、命名や説明を工夫することで効果を発揮します。

Habitz では、オンボーディング画面での大きなヒーロー画像と、アプリカードでの小さなイラストの2つの方法でイラストを使用しています。この小さなイラストは、オプションでアクセントカラーをつけることもできます。

### コンポーネントセットの作成
Kai は、ユースケースごとに2つのコンポーネントセットを作成することを望んでいます。そうすることで、デザイナーは自分の状況に合ったイラストを見つけやすくなります。それぞれのイラストに、用途と種類を表す名前を付けます。

ヒーローイラストを選択した状態で、ツールバーから「コンポーネントセットの作成」を選択します。右側のサイドバーで、「プロパティ」セクションのデフォルトのプロパティ（プロパティ1）を更新し、名前を「タイプ」に変更します。
小さいイラストの場合、Kai さんはアクセントカラーを考慮し、それぞれの名前に別の属性を追加したいと考えています。

https://help.figma.com/hc/en-us/articles/360039958934