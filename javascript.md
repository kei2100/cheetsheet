### 3項演算子でオブジェクトのフィールドを定義するイディオム

```js
let obj = {
  foo: "foo text",
  ...(condition ? { bar: "bar text" }: {}),
}
```
