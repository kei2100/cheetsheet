### mongoshell
```
# db
show dbs
use {db_name}

# collection
show collections

# document
db.{col_name}.count()

# slow operationを記録する
db.setProfilingLevel(0, { slowms: 100 })
db.system.profile.find().limit(10).sort( { ts : -1 } ).pretty()  // over 100ms なコマンドがsystem.profileコレクションに記録される（capped collection）
```

### query

```javascript
// 文字列の長さ n 以上で検索
{ "tel": { "$ne": null }, "$expr": { "$gt": [ { "$strLenCP": "$tel" }, 11 ] } }
```

### コレクション別ドキュメントサイズ平均ランキング

```js
db.getCollectionNames().map(function(col) {
    return {
        name: col,
        docSizeAvg: db[col].dataSize() / db[col].count()
    };
}).sort(function(a, b) { 
    return b.docSizeAvg - a.docSizeAvg;
}).map(function(obj, i) { 
    obj["rank"] = i + 1;
    return obj;
});
```
