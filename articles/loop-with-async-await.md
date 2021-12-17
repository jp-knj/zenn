---
title: "ループで考えるasync/await"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 配列の Promise を実行する

```javascript:api.js
const axios = require('axios');
const endpoint = 'https://rickandmortyapi.com/api/character';

const characterIds = [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

async function getCharacter(id) {
  return axios.get(`${endpoint}/${id}`).then((r) => r.data);
}

module.exports = { getCharacter, characterIds };
```

```javascript:for.js
let result = [];
(async function() {
  console.time('get character')
  for(let i = 0; i < characterIds.length; i++){
    const character = await getCharacter(characterIds[i]);
    results.push(`${character.id}:${character.name}`);
  }
  console.timeEnd('get character')
  console.log(results);
})();
// RESULT: 1.177s
```

```javascript:for.js
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
// RESULT: 12.563ms 
```