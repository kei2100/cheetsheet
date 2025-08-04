## コマンド

* データベース一覧: \l
  * OR `select * from pg_database;`  
* データベース切り替え: \c <DB名>
* テーブル一覧: \d
* テーブル内容: \d <テーブル名>
* テーブル内容コメント付き: \d+ <テーブル名>
* テーブルの権限: \z <テーブル名：指定しないと一覧>
* ページャーの on/off: \pset pager
* MySQL でいうところこの \G 表示に切り替え: \x
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

## JSON エクスポート

```sql
SELECT json_agg(row_to_json(people))
FROM people
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

## ダブルクォートのルール

- user, using など予約後と衝突する場合
- 大文字が含まれる場合。ダブルクォートで囲まないと postgresql は提供された識別子を小文字に変換しようとする
- 空白やハイフンやスラッシュなどの特殊文字が含まれている場合


## CREATE DATABASE

```
-- app ユーザの作成
CREATE USER app WITH PASSWORD '<password>';

-- DB の作成
CREATE DATABASE "my-db"

-- データベースの所有権を app ユーザに付
ALTER DATABASE "my-db" OWNER TO app;

-- app ユーザに必要な権限を付与
GRANT ALL PRIVILEGES ON DATABASE "my-db" TO app;

# app ユーザーで入り直し
psql -U app -p 5432 -h <database host> -d my-db

-- public スキーマのすべてのテーブルに対する権限
GRANT USAGE, CREATE ON SCHEMA public TO app;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO app;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO app;

-- 新しく作成されるオブジェクトに対するデフォルト権限を設定
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON FUNCTIONS TO app;
```
