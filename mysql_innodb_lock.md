# InnoDB ロックアーキテクチャ

InnoDB のロックアーキテクチャについてのメモ

※ 本文中例示される emp（従業員）テーブルの内容は以下の通り

```sql
CREATE TABLE `emp` (
  `empno` int(11) NOT NULL DEFAULT '0',
  `ename` varchar(255) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `job` varchar(255) COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `mgr` int(11) NOT NULL DEFAULT '0',
  `hiredate` date NOT NULL DEFAULT '1000-01-01',
  `sal` decimal(10,2) DEFAULT NULL,
  `comm` decimal(10,2) DEFAULT NULL,
  `deptno` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`empno`),
  KEY `idx_job` (`job`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

## トランザクション分離レベルと読み取り一貫性

各トランザクション分離レベルと読み取り一貫性を以下表で整理する。
（する => 発生する、しない => 発生しない)

| 分離レベル | ダーティーリード | ファジーリード | ファントムリード |
| ---- | ---- | ---- | ---- |
| READ UNCOMITED | する | する | する |
| READ COMITED | しない | する | する |
| REPEATABLE READ <br> (InnoDB のデフォルト) | しない | しない | する <br> (InnoDB ではしない) |
| SERIALIZABLE | しない | しない | しない |


* ダーティーリード ... Trx の COMMMIT していない変更を、他の Trx から参照できてしまう状態
* ファジーリード ... 2つの Trx (T1, T2) が開始し、片方の T1 があるレコードを「変更」して COMMIT したとき、その内容がまだ COMMIT していない T2 から参照できてしまう状態
* ファントムリード ... 2つの Trx (T1, T2) が開始し、片方の T1 があるレコードを「追加」または「削除」して COMMIT したとき、その内容がまだ COMMIT していない T2 から参照できてしまう状態

## ファントムリードを防ぐ

InnoDB では `REPEATABLE READ` 以上のトランザクション分離レベルを設定することで、ファントムリードを防ぐことができる。他の DBMS では `REPEATABLE READ` だとファントムリードが発生する、とされていることが多い。

```
# 通常 READ COMMITED では以下 T2 がコミットした変更が T1 から見えてしまっている

T1: READ_COMMITTED
T2: READ_COMMITTED

T1: QUERY: SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )

T2: UPDATE: INSERT INTO emp (empno, ename) VALUES (7785, 'steve')
(T2:UPDATE:COUNT=1)
T2: COMMIT

T1: QUERY: SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7785 steve null null null null null null )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )
```

```
# REPEATABLE_READ だとファントムリードは発生しない

T1: REPEATABLE_READ
T2: REPEATABLE_READ

T1: QUERY: SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno LOCK IN SHARE MODE
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )

T2: UPDATE: INSERT INTO emp (empno, ename) VALUES (7785, 'steve')
(T2:UPDATE:COUNT=1)
T2: COMMIT

1:QUERY:SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )
(1:QUERY)
```

また `REPEATABLE READ` で、SELECT に `LOCK IN SHARE MODE` 、あるいは `FOR UPDATE` を付与した `ロッキングリード` を行うと、SELECT した範囲に ファントムリードをしようとするトランザクションを待たすようになる。（待機させるトランザクションは `REPETABLE READ` でなくとも。）

```
# T2 は T1 がロッキングした範囲に INSERT することができず待機させられる
T1: REPEATABLE_READ
T2: READ_COMMITTED

T1: QUERY: SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno LOCK IN SHARE MODE
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )

T2:UPDATE:INSERT INTO emp (empno, ename) VALUES (7785, 'steve')
(T2:UPDATE:COUNT=0)
(T2:java.sql.SQLException: Lock wait timeout exceeded; try restarting transaction)
T2:ABORT

T1: QUERY: SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 ORDER BY empno
(empno ename job mgr hiredate sal comm deptno )
(7782 clark manager 7839 1981-06-09 2452.00 null 10 )
(7788 scott analyst 7566 1987-04-19 3002.00 null 20 )
```

## トランザクション分離性を実現させる2つの方式 （MVCC とロック）

InnoDB は、トランザクション分離性を実現するために以下2つの方式を使用している。

* ロック方式
  * 先行する SQL が `共有ロック` または `排他ロック` を取得し、他のトランザクションがロックを取得しようとするのを待たせる
* MVCC 方式 (Multi Version Concurrency Control)
  * SELECT 文で使われる
  * ロックを取得しないため、他トランザクションの共有ロックや排他ロックと競合しない
  * 他のトランザクションによる更新があった場合は、`UNDO` を用いて更新前のデータを見せる


InnoDB のロック方式は、さらに細かく `レコードロック`、`ネクストキーロック` などの方式に分けられ、トランザクション分離レベルごとに以下のように使い分けられる。

| 分離レベル | SELECT | SELECT 〜 LOCK IN SHARE MODE | DML (SELECT 〜 FOR UPDATE も同様)  |
| ---- | ---- | ---- | ---- | 
| READ COMMITTED  | 文単位の MVCC             | 共有レコードロック    | 排他レコードロック    |
| REPEATABLE READ | トランザクション単位の MVCC | 共有ネクストキーロック | 排他ネクストキーロック |
| SERIALIZABLE    | 共有ネクストキーロック      | 共有ネクストキーロック | 排他ネクストキーロック |

## InnoDB のレコードロックタイプとロックモード

InnoDB のレコードレベルのロックには、以下の複数の `ロックタイプ` がある

* `レコードロック`: インデックスレコードそのものに対するロックを表す
* `ギャップロック`: 2つの連続するインデックスレコード間に対するロックを表す
* `ネクストキーロック`: レコードロックとその手前のギャップロックを表す

以下の図は、それぞれのロックタイプを表す

* A: 7788 のレコードロック
* B: 7782 と 7788 の間のギャップロック
* C: ネクストキーロック（ A のレコードロックと B のギャップロックを足したもの）

```
# empno のプライマリインデックス

(empno)
7698   7782  7788  7839
  |-----|-----|-----| 
              A
         |-B-|
         |--C-|
```

ギャップロックやネクストキーロックのように範囲をロックするのはファントムリードを防止するため


また `ロックモード` には以下の種類がある

* 排他: 他トランザクションのすべてのロックをと競合する
* 共有： 他トランザクションの排他ロックと競合するが、共有ロックには互換する
* インテンション: 
  * インテンション排他ロック: インテンションロック同士では互換するが、排他と共有ロックには競合する
  * インテンション共有ロック: インテンションロック同士では互換するが、排他ロックには競合し、共有ロックには互換する

それぞれの競合互換関係を以下表に整理する

|                 | 排他 | インテンション排他 | 共有 | インテンション共有 | 
| --------------- | ---- | ---- | ---- | ---- |
| 排他             | 競合 | 競合 | 競合 | 競合 |
| インテンション排他 | 競合 | 互換 | 競合 | 互換 |
| 共有             | 競合 | 競合 | 互換 | 互換 |
| インテンション共有 | 競合 | 互換 | 互換 | 互換 |


## REPEATABLE READ におけるロックの範囲

### プライマリインデックスに対する等価検索、 IN 検索

```sql
SELECT * FROM emp WHERE empno = 7788 FOR UPDATE 

-- `FOR UPDATE` が付与されているので、 `WHERE empno = 7788` の DML と同様のロックとなる
```

* 7788 のレコードロックを取得する
* ファントムリードが発生する余地はないためギャップロックは発生しない

```
(empno)
7698   7782  7788  7839
  |-----|-----|-----| 
              L
```

* IN 検索は等価検索を複数回実行するのと同じ

```sql
SELECT * FROM emp WHERE empno IN (7782, 7788) FOR UPDATE
```

```
(empno)
7698   7782  7788  7839
  |-----|-----|-----| 
        L     L
```

### プライマリインデックスに対する範囲検索

ここからTODO

### TODO memo

INSERT でインテンションギャップロック（排他？）
インテンションロックの発生条件が不明

### デッドロックするクエリの例

空振りの DELETE INSERT 
回避策

### appendix
INSERT は記述した順に
IN 句は
    
