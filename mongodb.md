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
