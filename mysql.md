## user, grant

```sql
CREATE USER app@'%' IDENTIFIED BY '<password>';
GRANT SELECT, INSERT, UPDATE, DELETE ON `<schema>`.* TO app@'%';

CREATE USER migration@'%' IDENTIFIED BY '<password>';
GRANT ALTER, CREATE, DROP, INDEX, INSERT, UPDATE, DELETE, SELECT ON <schema>.* TO migration@'%';

CREATE USER admin@'%' IDENTIFIED BY '<password>';
GRANT ALL ON `<schema>`.* TO admin@'%';
```

```sql
-- 特定のテーブル以外 GRANT
REVOKE ALL PRIVILEGES ON db.* FROM user@localhost;  

SELECT CONCAT("GRANT UPDATE ON db.", table_name, " TO user@localhost;")
FROM information_schema.TABLES
WHERE table_schema = "YourDB" AND table_name <> "table_to_skip";
```

## 文字列検索では後続の空白は無視してマッチするの注意

```sql
mysql> select 'a    ' = 'a';
+---------------+
| 'a    ' = 'a' |
+---------------+
|             1 |
+---------------+
1 row in set (0.00 sec)
```

## TAB が含まれる文字列の検索

```sql
mysql> select "text\ttext" like concat("%", CHAR(9), "%");
+---------------------------------------------+
| "text\ttext" like concat("%", CHAR(9), "%") |
+---------------------------------------------+
|                                           1 |
+---------------------------------------------+
1 row in set (0.00 sec)
```

## 接続数

```sql
mysql> show status like 'Threads_connected';
```

```sql
mysql> show processlist;
```

## テーブルサイズの概算

```sql
SELECT  
    table_name, 
    engine, 
    table_rows,
    floor((data_length)/1024/1024) AS data_length_MB,  # 主キー領域を含むテーブルサイズ
    floor((index_length)/1024/1024) AS secondary_indices_MB, # セカンダリインデックスのサイズ
    floor((data_length+index_length)/1024/1024) AS sum_MB
FROM 
    information_schema.tables  
WHERE
    table_schema=database()  
ORDER BY
    sum_MB DESC;  
```
