## user, grant

```sql
CREATE USER app@'%' IDENTIFIED BY '<password>';
GRANT SELECT, INSERT, UPDATE, DELETE ON `<schema>`.* TO app@'%';

CREATE USER migration@'%' IDENTIFIED BY '<password>';
GRANT ALTER, CREATE, DROP, INDEX, INSERT, UPDATE, DELETE, SELECT ON <schema>.* TO migration@'%';

CREATE USER admin@'%' IDENTIFIED BY '<password>';
GRANT ALL ON `<schema>`.* TO admin@'%';
```

## 文字列検索では後続の空白は無視してマッチするの注意

```
mysql> select 'a    ' = 'a';
+---------------+
| 'a    ' = 'a' |
+---------------+
|             1 |
+---------------+
1 row in set (0.00 sec)
```
