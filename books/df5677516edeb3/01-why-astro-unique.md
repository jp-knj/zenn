---
title: "Astroの設計思想と誕生の背景"
---

Astroは"必要なJavaScriptだけを届ける"という明快な目標を掲げて誕生しました。本書では、その内部構造と設計判断を自作するつもりで分解し、読み手が自らのコードに置き換えて再現できるレベルまで掘り下げていきます。第1章はAstroが生まれた背景と根底にある思想を整理します。

この章を読み終えるころには、Astroの各機能が単なる実装上の都合ではなく、歴史的経緯と課題解決の必然から導かれたものであることが見えてくるはずです。

## なぜ"速い"はずのサイトが遅いのか

2020年時点、モバイルウェブページは75%で **約 3.7 MB** のリソースを転送しています。画像が最大の重いリソースになりますが、JavaScriptも2番目に重いリソースです。合計サイズを大きく押し上げています（HTTP Archive Web Almanac 2020 [Page Weight](https://almanac.httparchive.org/en/2020/page-weight/#page-weight) 章）。  
3 G相当（下り400 kbps）の帯域で3.7 MBを取得すると、*Time to Interactive* は **およそ 8 秒**、*Interaction to Next Paint (INP)* も400 msを超えます。

ユーザーはタップしてから画面が反応するまで、はっきり体感できる待ち時間が発生するわけです。

> *“JavaScript is the most expensive resource we send to mobile phones, because it can delay interactivity in a way that other resources don’t.”*
> — Addy Osmani, *The Cost of JavaScript (2018)*【[GitNation GameSnacks トーク](https://gitnation.com/contents/making-bite-sized-web-games-with-gamesnacks)[1]】

Google Chrome Labsも開発者向けガイドで"[Reduce JavaScript payloads with tree shaking](https://web.dev/articles/reduce-javascript-payloads-with-tree-shaking)"と題し、

> “JavaScript is an expensive resource to process. … Byte for byte, JavaScript is more expensive than other types of resources.”
> と警告しています。([web.dev][2])

こうした数字とこのような情報が示すのは**大規模なJavaScriptバンドルが、ネットワークとCPUの両面でユーザー体験を阻害している**という事実です。

この"肥大化問題"の全体像を整理しると、次のような要素が浮かび上がります。

## なぜ、Islands が必要だったのか

2020年前後、ReactやVueが成熟し、シングルページアプリケーション (SPA) が標準解のように扱われる一方で、初期ロードに必要なバンドルが肥大化し続けていました。モバイル環境では特に顕著で、数Mバイト規模のJavaScriptがネットワーク帯域とバッテリーを圧迫し、ユーザー体験を犠牲にしていました。

SSRやSSGを導入したNext.jsやGatsbyも、最終的にはページ全体をハイドレートするために大量のJavaScriptを必要とする構造から抜け出せませんでした。"最適化されているはずの静的サイトが、なぜこんなにJSを抱えているのか"という疑問が、多くの開発者で共有され始めた時期でもあります。

> ハイドレート対象 = 送る JS 量

ViteやBunがどれだけビルドと実行を速くしても、"DOM全体をReact/Vueが取り込んで再描画する"というSPA型ハイドレーションを選ぶ限り、ブラウザには依然としてフレームワーク本体＋ページ単位のJSが届きます。結果として実行時メモリやCPU利用率は高止まりし、モバイル体験のボトルネックを取り除けません。
Viteが得意なのは"ビルド時間と開発者体験の高速化"であって、生成されるバンドル総量やクライアントのハイドレーション負荷は、採用するUIフレームワークの構造（完全ハイドレーションか部分ハイドレーションか）に依存します

## 着想 - Snowpack

Preact作者のJason MillerはブログでIslands Architectureを提示した。彼は、"まずHTMLを送り、必要な部位だけを後からハイドレートする"という逆転の発想で、モバイル体験を劇的に軽くする道筋を示した。

同じ頃、Fred SchottがSnowpackというESMベースのビルドツールを公開した。開発中はバンドラーを介さず、ブラウザがネイティブモジュールを直接読む――この実験は"不要な変換を減らすことで開発も本番も速くなる"ことを証明し、のちにAstroの発想へと結実する。

## Astro の三つの設計原則

Astroの公式サイト"Why Astro?"に掲げられる三原則は次のとおりです。

* **コンテンツ中心** : MarkdownやCMSから取得したデータを第一級市民として扱い、UIロジックよりも優先する。
* **サーバーファースト** : 可能な限りの処理をサーバーで完了させ、クライアントは本当に必要な瞬間までJavaScriptをダウンロードしない。
* **デフォルトで高速** : 特別な設定を追加しなくても、高速なサイトが手に入ることを前提にする。

この方針を支える具体的な仕組みが、`client:*` ディレクティブを介した選択的ハイドレーションです。開発者が明示的に指示しない限り、コンポーネントのJavaScriptは出力されず、HTMLのみを配信します。

## 同時代フレームワークとの対話と Astro の立ち位置

React Server Componentsが掲げる"送るJSの削減"、Remixが推進するProgressive Enhancement、Qwikが実装するResumability――どれも過剰なハイドレーション負債からの脱却を目指している。しかしAstroは早期に"まず静的HTMLを届け、必要な部分だけ動的化する"という単純かつ実用的な解を提示した点で際立つ。皆同じ問題を別解で解いてるように見える。

## まとめと次章への橋渡し

JavaScript肥大化がもたらした問題から出発し、アイランドアーキテクチャを核とするAstroの設計思想を概観しました。ここで得た歴史的背景と三原則を頭に置くことで、次章以降の技術解説が"なぜそうするのか"という問いに自然と紐づくはずです。次章では、アイランドアーキテクチャをどのようにコードで実装し、ビルド時に静的HTMLとプレースホルダーを生成するのかを具体的に見ていきます。

[1]: https://gitnation.com/contents/making-bite-sized-web-games-with-gamesnacks "Making “Bite-Sized” Web Games with GameSnacks by Alex Hawker"
[2]: https://web.dev/articles/reduce-javascript-payloads-with-tree-shaking "Reduce JavaScript payloads with tree shaking  |  Articles  |  web.dev"
