## コマンド

* データベース一覧: \l
  * OR `select * from pg_database;`  
* データベース切り替え: \c <DB名>
* テーブル一覧: \d
* テーブル内容: \d <テーブル名>
* テーブルの権限: \z <テーブル名：指定しないと一覧>
* ページャーの on/off: \pset pager
* default privileges の表示
  ```
  \ddp 
  
                 Default access privileges
       Owner      | Schema |   Type   | Access privileges 
  ----------------+--------+----------+-------------------
   role_x         |        | function | =X/role_x
   role_x         |        | sequence | 
   role_x         |        | table    | 
   role_x         |        | type     | =U/role_x
  ```
* TBD
  
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

## 無名コードブロック

DOを使うと無名コードブロック（一時的な無名関数）を実行することができる

```
DO $$ BEGIN
  -- dev-dbのみ実行
  IF current_database() = 'dev-db' THEN
    UPDATE foo SET bar = 'bar' WHERE bar = 'baz';
  END IF;
END $$;
```
