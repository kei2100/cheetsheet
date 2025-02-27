## コマンド

* データベース一覧: \l
  * OR `select * from pg_database;`  
* データベース切り替え: \c <DB名>
* テーブル一覧: \d
* テーブル内容: \d <テーブル名>
* テーブルの権限: \z <テーブル名：指定しないと一覧>

## SHOW

|項目|内容|
|--|--|
|max_connections|最大接続数|
|superuser_reserved_connections|管理用接続数。max_connectionsからこれを引いた値が利用可能な接続数|


## 文字コードの確認

`=> \encoding`
あるいは
`$ psql -l`

## 現在のユーザー

```
SELECT CURRENT_USER
```
