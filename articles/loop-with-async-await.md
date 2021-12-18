---
title: 【初学者向け】非同期で考えるとループ処理がわかる？
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['javascript']
published: false
---

# はじめに 
この記事では、`for`,`forEach`,`for..in`,`map`の違いを非同期処理を通して、解説します。   

## 📚 非同期処理の準備
- [axios](https://github.com/axios/axios)
  axiosはPromiseベースのHTTPClientライブラリでGETやPOSTのHTTPリクエストを使ってサーバからデータの取得、データへのデータ送信を行い、この記事では、データ(JSON)を取得するために利用

- [The Rick and Morty API](https://rickandmortyapi.com/)
  こちらのデータ(API)を使用

```javascript:api.js
const axios = require('axios');
const endpoint = 'https://rickandmortyapi.com/api/character';

const characterIds = [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
async function getCharacter(id) {
  return axios.get(`${endpoint}/${id}`).then((result) => result.data);
}

module.exports = { getCharacter, characterIds };
```
axiosを読み込み、endpoint[^1]を定義します。

*The Rick and Morty API* のデータを取得するために `characterIds` と `getCharacter関数` を追加し、exportしています。

[^1]: endpoint(エンドポイント)とは、指定されたリソースに対して与えられた固有の一意な URI のこと。この記事では `api.js` にアクセスされる `https://rickandmortyapi.com/api/character`(URL) のことです。

これから、本題に進みます
## 非同期で for について考える
```javascript:for.js
const { getCharacter, characterIds } = require('./api')

let results = [];
(async function() {
  console.time('get character') // タイマーの開始する
  
  for(let i = 0; i < characterIds.length; i++){
    const character = await getCharacter(characterIds[i]);
    results.push(`${character.id}:${character.name}`);
  }
  console.timeEnd('get character') // タイマーの停止する
  
  console.log(results);
})();
// TIME: 1.177s 
// RESULTS: [
// '1:Rick Sanchez',
// '2:Morty Smith',
// '3:Summer Smith',
// '4:Beth Smith',
// '5:Jerry Smith',
// '6:Abadango Cluster Princess',
// '7:Abradolf Lincler',
// '8:Adjudicator Rick',
// '9:Agency Director',
// '10:Alan Rails'
// ]
```

即時関数[^2]の本体に経過時間を図るため`console.time`[^3] と`console.timeEnd`[^4]を利用します。
実行すると、経過時間は *1.177s* かかり、体感で遅いと分かります。

では、forEach の場合はどうなるのでしょうか？

[^2]: 関数を定義すると同時に実行するための構文。関数の定義と実行するためコードを分別せずに即時関数にしております。
[^3]: 測りたい処理の開始時にconsole.time()を呼び出す。このとき引数にラベル名を渡す必要がある。
[^4]: 計測終了したいタイミングでconsole.timeEnd()を呼び出す。このとき引数には同じラベル名を渡す。

## 非同期で forEach について考える
```javascript:forEach.js
let result = [];
(async function() {
  console.time('get character')
  characterIds.forEach(async(id) => {
    const character = await getCharacter(id);
    results.push(`${character.id}:${character.name}`);
  });
  console.timeEnd('get character')
  console.log(results);
})();

// TIME: 12.563ms 
// RESULTS: []
```

実行時間は `12,563ms`とかなり早かったのですが、`empty` が戻ってきました。
実は`async/await`(非同期処理) をするとき forEach は使えないのです。

### なぜ、非同期では forEach は使えないでしょうか？
forEach[^5]のコールバックの返り値は`undefined`になります。なので、引数に`async関数`(Promise)を入力された`await`されることもなく、`empty`になる

[^5]: [forEach](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)

では非同期をするにはどうすればよいのでしょうか？

## 非同期で map と Promise.all について考える
```javascript:map.js
const { getCharacter, characterIds } = require("./api");

(async function () {
    console.time('get character')

    const mapResult = characterIds.map((id) => {
        return getCharacter(id);
    });

    const results = await Promise.all(mapResult);
    console.timeEnd('get character')
    console.log(results);
})();

// TIME : 170.563ms 
// RESULTS: [
//   {
//     id: 1,
//     name: 'Rick Sanchez'
//     // ...
//   },
//   // ...
// ]
```