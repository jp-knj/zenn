---
title: "JavaScriptの基礎まとめ【前半】"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "初学者"]
published: true
---
# はじめに
JavaScriptの初級者が実務で学んだ事や、公式ドキュメントや技術書でインプットした内容を記事としてアウトプットさせていただいてます。もし、不適切な箇所があればご指摘よろしくお願いします。

# thisについて
　this は　オブジェクトを参照します。ですが、コードの文脈によって挙動が変動してしまいます。

- メソッド, 関数, コンストラクタ, グローバルオブジェクト を参照する
- apply, call, bind でオブジェクトを指定する

## メソッド, 関数, コンストラクタ を参照する
 この `this` は `method` を参照していますね。
 . (ドット) の前にあるオブジェクトを参照する性質を持っています。

```javascript:script.js
/**
 * メソッドや関数の場合
 */
let method  = {
  value: 10,
  show() {
    console.log(this.value);
  }
}
method.show(); // 10

/**
 * コンストラクタの場合
 */
class User {
  constructor( name, age) {
    this.name = name;
    this.age = age;
  }
  getName() {
    return this.name;
  }
  getAge() {
    return this.age;
  }
}

//インスタンスを生成する
const kenji = new User('kenji', 20);

console.log(kenji.getName()); // kenji
console.log(kenji.getAge());  // 20
```

例外もあります！ネストされた `this` は window (グローバルオブジェクト)を参照します
蟻地獄のようなコードは見たくありませんね。
```javascript:script.js
const obj = {
  func() {
    console.log(this);
    const func2 = function(){
      console.log(this);
      const func3 = function(){
        console.log(this);
      }();
    }();
  }
}

obj.func();　// window
```
### グローバルオブジェクト を参照する
 この `show()` は　. (ドット) がありません。ですが、実際は `window.show()` という風に呼びされており、window (グローバルオブジェクト)を参照していますね。
```javascript:script.js
foo = "foo";
const show = function() {
  console.log(this.foo);
}

show();
```

参考にした記事
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/this
https://qiita.com/takkyun/items/c6e2f2cf25327299cf03
# apply, call, bind について
 `call()` は `this` のオブジェクトを指定できる関数になっています。
```javascript:script.js
function sayHi(){
    let hi = `Hi, ${this.name}`;
    console.log(hi);
}
let obj = {name : "kenji"}

sayHi.call(obj)
```
 `apply()` も `this` のオブジェクトを指定できる関数になっています。では、`call()` と `apply()` はどういう風に使い分けるのでしょうか？  
それは簡単で引数が単数なら `call()` を使い、 引数が複数なら`apply()`を使います。
```javascript:script.js
const array = [ 1, 2, 3, 4];
sayHi.apply(obj, array)
```
`bind()` は `this` のオブジェクトを束縛できる関数になっています。

```javascript:script.js
/**
 *  bind()を使用しない場合
 */

let obj = {
  id : 1,
  show(){
    console.log(this);
    setTimeout(function() {
      console.log(this)
    }, 1000);
  }
}

obj.show();
// obj
// undined // グローバルオブジェクトを参照しています。

/**
 *  bind()を使用した場合
 */

let obj = {
  id : 1,
  show(){
    console.log(this);

    setTimeout(function() {
      console.log(this)
    }.bind(this), 1000);
  }
}

obj.show();
// obj
// obj // obj をポイントするようになる
```

# アロー関数 について
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

参考にした記事
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Arrow_functions
## クロージャー
> クロージャは、組み合わされた（囲まれた）関数と、その周囲の状態（レキシカル環境）への参照の組み合わせです。言い換えれば、クロージャは内側の関数から外側の関数スコープへのアクセスを提供します。JavaScript では、関数が作成されるたびにクロージャが作成されます。[引用元](https://developer.mozilla.org/ja/docs/Web/JavaScript/Closures)

ボク自身、この説明では理解できなかったのでどういう性質を持っているか、コードで見ていきました。
親の戻り値の関数の範囲( レキシカルスコープ )にある 変数`counter`は外側から参照されない仕組みになっているそうです。関数内と関数外の`counter`は別々の変数になっています。

```javascript:script.js
// 即時関数を使用しています
let increment = (function(){
  let counter = 0; 
  return function(){
    counter += 1;
    console.log(counter);
  }
})();

increment.counter = 2; // 関数外から変数の値を変更する
increment(); // 1
increment(); // 2
increment(); // 3
```

## 後半へ
