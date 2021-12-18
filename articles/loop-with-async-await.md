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
let result = [];
(async function() {
  console.time('get character') // タイマーの開始する
  
  for(let i = 0; i < characterIds.length; i++){
    const character = await getCharacter(characterIds[i]);
    results.push(`${character.id}:${character.name}`);
  }
  console.timeEnd('get character') // タイマーの停止する
  
  console.log(results);
})();
// Time : 1.177s 
// Result : []
```

> タイマー
> 特定の操作にかかる時間を計るため、タイマーを設定することができます。タイマーを開始するには、引数で名前を指定して console.time() メソッドを呼び出します。タイマーを停止して経過時間をミリ秒単位で取得するには、タイマーの名前をまた引数で指定して console.timeEnd() メソッドを呼び出します。ページあたり最大 10,000 個のタイマーを同時に動かすことができます。

引用 : [mozilla](https://developer.mozilla.org/ja/docs/Web/API/console#timers)


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

// TIME : 12.563ms 
// RESULT: []
```

実行時間は `12,563ms`とかなり早かったのですが、`empty` が戻ってきました。
これは記述のミスではなく、`async/await`(非同期処理) をするとき forEach は使えないのです。

forEachは何が来ようが、コールバックの返り値を無視します。結果、async関数が生成したPromiseも無視されて、awaitされることもなく進んでしまいます。

map と Promise.all
```javascript:map.js
(async function() {
  console.time('get character')
  
  const mapResult = characterIds.map((id) => {
    return getCharacter(id);
  });
  
  const results = await Promise.all(mapResult);
  console.timeEnd('get character')
  console.log(results);
})();
// TIME : 170.563ms 
// RESULT: [{
      "id": 1,
      "name": "Rick Sanchez",
      "status": "Alive",
      "species": "Human",
      "type": "",
      "gender": "Male",
      "origin": {
        "name": "Earth",
        "url": "https://rickandmortyapi.com/api/location/1"
      },
      "location": {
        "name": "Earth",
        "url": "https://rickandmortyapi.com/api/location/20"
      },
      "image": "https://rickandmortyapi.com/api/character/avatar/1.jpeg",
      "episode": [
        "https://rickandmortyapi.com/api/episode/1",
        "https://rickandmortyapi.com/api/episode/2",
        // ...
      ],
      "url": "https://rickandmortyapi.com/api/character/1",
      "created": "2017-11-04T18:48:46.250Z"
    },
    // ...
]
```