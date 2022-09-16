---
title: ""
emoji: "🐈"
type: "idea" 
topics: []
published: false
---
可読性の高いテストコードや TypeScript をしようしたテスト戦略を考えて、要約した記事となります。
ですが、以下の内容には誤りがある可能性があります。もし何かあれば、ご教示頂けると幸いです。

## はじめに

## テストをどこに置くのか
複数のOSSプロジェクト、業務で良くみるテストのディレクトリは大まかに3つに分けられると思います。
#### Case1.
- プロジェクトルートに testsディレクトリ を配置する方法
  - srcディレクトリの構造と同じ構造で複製する。
  - テスト対象のモジュールと対応する場所にテストコードを配置する。(Java, Python, Ruby ではこのファイル配置が多い)
```bash
.
├── tests
│   ├── sample.test.js
│   └── directory
│       └── sample.test.js
└── src
    ├── sample.js
    └── directory
        └── sample.js
```
#### Case2.
パターン2. ソースコードの各ディレクトリに __tests__ や tests ディレクトリを作り、同階層のモジュールのテストをそのディレクトリの中に配置する
```bash
.
├── domain
│   ├── test
│   │   └── sample.test.js
│   └── directory
│       └── sample.js
└── usecase 
    ├── test
    │   └── sample.test.js
    └── directory
        └── sample.js
```
#### Case3.
- ソースコードファイルの隣接にテストファイルを配置する
## テストレベルについて

## Rustのテストを参考に

https://doc.rust-jp.rs/book-ja/ch11-00-testing.html
## テスト戦略

## さいごに
