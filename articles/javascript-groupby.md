---
title: "[JavaScript] アイテムを選り分ける、groupBy"
emoji: "🤏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript"]
published: true
---
# はじめに
Baseline 2024 より、Newly Available（主要なブラウザで使用可）となった機能の一つ、`Object.groupBy()` についての記事です。
2024 年に Baseline に組み込まれた機能の一覧は、以下の記事を参照しています。
https://web.dev/series/baseline-newly-available?hl=ja

# 概要
反復可能な要素 `items` と、任意のコールバック関数 `callbackFn` を引数に取ります。

```js
Object.groupBy(items, callbackFn)
```

この関数が実行されると `items` の各要素に対して `callbackFn` が実行され、この関数の返り値によってグループに選り分けられます。この返り値は文字列（もしくはシンボル）である必要があり、それ以外の場合は文字列に変換されて返されます。

例として、以下のようなデータを考えます。
```js
const users = [
  { name: "Tom", gender: "male", age: 62 },
  { name: "Emily", gender: "female", age: 36 },
  { name: "William", gender: "male", age: 61 },
  { name: "Anne", gender: "female", age: 42 },
  { name: "Cameron", gender: "female", age: 52 },
  { name: "Leonardo", gender: "male", age: 50 },
]
```

性別でグルーピングして。それぞれの配列として取得してみます。
結果は以下の通り。
```js
const result = Object.groupBy(users, (user) => user.gender);

// {
//   "male": [
//       { "name": "Tom", "gender": "male", "age": 62 },
//       { "name": "William", "gender": "male", "age": 61 },
//       { "name": "Leonardo", "gender": "male", "age": 50 }
//   ],
//   "female": [
//       { "name": "Emily", "gender": "female", "age": 36 },
//       { "name": "Anne", "gender": "female", "age": 42 },
//       { "name": "Cameron", "gender": "female", "age": 52 }
//   ]
// }
```

より高度な例も挙げておきます。`age` の値を、40歳未満、40歳以上60歳未満、60歳以上の3つのグループに分割する例です。
```js
const groupAgeCallback = (user) => {
  if(user.age < 40) {
    return "age < 40";
  } else if(user.age < 60) {
    return "40 <= age < 60";
  } else {
    return "60 <= age";
  }
}

const result = Object.groupBy(users, groupAgeCallback);

// {
//     "60 <= age": [
//         { "name": "Tom", "gender": "male", "age": 62 },
//         { "name": "William", "gender": "male", "age": 61 }
//     ],
//     "age < 40": [
//         { "name": "Emily", "gender": "female", "age": 36 }
//     ],
//     "40 <= age < 60": [
//         { "name": "Anne", "gender": "female", "age": 42 },
//         { "name": "Cameron", "gender": "female", "age": 52 },
//         { "name": "Leonardo", "gender": "male", "age": 50 }
//     ]
// }
```

---

注意点として、渡した `items` 内の各オブジェクトと `groupBy` が返す結果内の各オブジェクトは**同一であり（コピーではない）**、`callbackFn` にオブジェクトを変更するようなコードが含まれる場合、`groupBy` を使用することで `items` と結果の両方に変更が加わることがあります。

極端な例ですが、`callbackFn` 内で `age` を `50` に変更するコードを書いたとします。
```js
const result = Object.groupBy(users, (user) => {
  user.age = 50;
  return user.gender;
});
```

これを実行した直後の `users` と `result` の結果が以下の通りです。
```js
console.log(users)

// [
//   { "name": "Tom", "gender": "male", "age": 50 },
//   { "name": "Emily", "gender": "female", "age": 50 },
//   { "name": "William", "gender": "male", "age": 50 },
//   { "name": "Anne", "gender": "female", "age": 50 },
//   { "name": "Cameron", "gender": "female", "age": 50 },
//   { "name": "Leonardo", "gender": "male", "age": 50 }
// ]

console.log(result)

// {
//     "male": [
//         { "name": "Tom", "gender": "male", "age": 50 },
//         { "name": "William", "gender": "male", "age": 50 },
//         { "name": "Leonardo", "gender": "male", "age": 50 }
//     ],
//     "female": [
//         { "name": "Emily", "gender": "female", "age": 50 },
//         { "name": "Anne", "gender": "female", "age": 50 },
//         { "name": "Cameron", "gender": "female", "age": 50 }
//     ]
// }
```

`result` だけでなく、データ元の `users` にも変更が及んでいることが分かります。

これを防ぐために、**分割代入引数**によって引数を受け取ることができます。

https://typescriptbook.jp/reference/functions/destructuring-assignment-parameters

```js
const users = [
  { name: "Tom", gender: "male", age: 62 },
  { name: "Emily", gender: "female", age: 36 },
  { name: "William", gender: "male", age: 61 },
  { name: "Anne", gender: "female", age: 42 },
  { name: "Cameron", gender: "female", age: 52 },
  { name: "Leonardo", gender: "male", age: 50 },
]

const result = Object.groupBy(users, ({ age, gender }) => {
  age = 50;
  return gender;
});

console.log(users);

// [
//   { name: "Tom", gender: "male", age: 62 },
//   { name: "Emily", gender: "female", age: 36 },
//   { name: "William", gender: "male", age: 61 },
//   { name: "Anne", gender: "female", age: 42 },
//   { name: "Cameron", gender: "female", age: 52 },
//   { name: "Leonardo", gender: "male", age: 50 },
// ]

console.log(result);

// {
//   "male": [
//       { "name": "Tom", "gender": "male", "age": 62 },
//       { "name": "William", "gender": "male", "age": 61 },
//       { "name": "Leonardo", "gender": "male", "age": 50 }
//   ],
//   "female": [
//       { "name": "Emily", "gender": "female", "age": 36 },
//       { "name": "Anne", "gender": "female", "age": 42 },
//       { "name": "Cameron", "gender": "female", "age": 52 }
//   ]
// }
```

# 参照
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy