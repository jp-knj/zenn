---
title: ""
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# はじめに
JavaScriptの初級者が実務で学んだ事や、公式ドキュメントや技術書でインプットした内容を記事としてアウトプットさせていただいてます。もし、不適切な箇所があればご指摘よろしくお願いします。

## this

参考にした記事
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/this
https://qiita.com/takkyun/items/c6e2f2cf25327299cf03
## apply, call, bind
引数に配列ならapply
そうでないならcall

## アロー関数
適切にアロー関数を使用すると、可読性を上げる有用的な機能になります。ボクのような初級者でも目にしたことのある関数ですね。

**アロー関数の記述関して**
```javascript:script.js
/**
 * アロー関数での記述に関して (syntax)
 */
let arrowFunction;
arrowFunction = () => "Hello, World";
// 引数 (argument) が単数の場合は () を省略できます
arrowFunction = argument => "Hello, World";
// 引数 (argument) が複数の場合は () を省略できません
arrowFunction = (argument1, argument2) => "Hello, World";

// 関数内 (arrowFunction) のコードが２行以上ある場合、
// {} や return を記述しなければなりません
arrowFunction = (argument1, argument2) => {
    console.log(argument1 + argument2); // 1行目
    return argument1 + argument2;       // 2行目 
}
```

### アロー関数 と 通常関数 の動きの違いに関して

通常関数 　→ apply, call, bind →　指定できる
アロー関数 → apply, call, bind →　無視される

参考にした記事
https://qiita.com/suin/items/a44825d253d023e31e4d
## クロージャー

## スプレッド構文

## 分割代入

## テンプレートリテラル

## Callback 関数

## Promise 関数

## Async / Await 関数
