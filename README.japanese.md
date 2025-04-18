# `export` の範囲を制限する仕様の提案

java などの class では `private`, `public` に加え、`protected` 修飾子により適切なカプセル化が行える。
しかし `export` は、export するかしないかの 2 択しかない。

よって、ファイル分割するなら `export` するしかなく、 export されたものは他のすべてのファイルから参照可能となり、他のすべてのファイルはその export されたメンバを理解し適切に扱わなければならない状況を強制させられる。
逆に `export` しないなら、ファイル分割できず肥大化するしかなくなる。

class を用いないことも一般的にあり得る javascript では、`protected` のサポートではなく、`export` の仕様拡張を求めたい。

# 仕様案

`export` に optional で、公開範囲のパスを配列で渡せるようにする

```javascript
export['path1', 'path2', ...] const foo;
```

# 仕様案の例

同じディレクトリ内のすべてのファイルから import 可能とする場合

```javascript
export['./*'] const foo;
```

一つ上のディレクトリ内のすべてのファイルから import 可能とする場合

```javascript
export['../*'] const foo;
```

このディレクトリより下のすべてのファイルから import 可能とする場合

```javascript
export['../**'] const foo;
```

同じディレクトリ内の bar.js からのみ import 可能とする場合

```javascript
export['./bar.js'] const foo;
```

組み合わせ

```javascript
export['../baz.js', './*'] const foo;
```

# 期待される効果

class を用いないことも一般的な javascript において、適切なカプセル化を実現できるようになる。

例：

```
app/features/counter/lib.js;
  //ライブラリとの接点、カプセル化したい（ビジネスロジックからは参照されたくない）

app/features/counter/counter.js;
  //カウンタロジック実装、lib.js を使う

app/business.js;
  //ビジネスロジック実装、counter.js を使う
```

---

app/features/counter/lib.js

ライブラリのインポートはここでだけ

```javascript
import storage from 'something-lib';

const count = storage.create('key');

// 同一ディレクトリ内にのみエクスポートしカプセル化
export['./*'] const getCount = () => count.get();
export['./*'] const setCount = (newValue) => count.set(newValue);
```

---

app/features/counter/counter.js

カウンタの機能を実装

```javascript
import { getCount as get, setCount } from "./count.js";

export const getCount = () => get();
export const addCount = () => setCount(get() + 1);
export const resetCount = () => setCount(0);
```

---

app/business.js

```javascript
import { getCount, addCount, resetCount } from "./features/counter";

// 以下はインポートできない、ビジネスロジックでは以下の仕様を把握する必要がない
import { getCount, setCount } from "./features/counter/lib"; // error
```
