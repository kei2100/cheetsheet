# InnoDB ロックアーキテクチャ

InnoDB のロックアーキテクチャについてのメモ。

本文中例示される emp（従業員）テーブルの内容は以下の通り。

* empno がユニークなプライマリインデックス
* job に非ユニークなセカンダリインデックス

が定義されている。

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

InnoDB のレコードレベルのロックには、以下の複数の `ロックタイプ` がある。

* `レコードロック`: インデックスレコードそのものに対するロックを表す
* `ギャップロック`: 2つの連続するインデックスレコード間に対するロックを表す
* `ネクストキーロック`: レコードロックとその手前のギャップロックを表す

以下の図は、それぞれのロックタイプを表す。

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

ギャップロックやネクストキーロックのように範囲をロックするのはファントムリードを防止するためにある。
ギャップロック

また `ロックモード` には以下の種類がある。

* 排他: 他トランザクションのすべてのロックをと競合する
* 共有： 他トランザクションの排他ロックと競合するが、共有ロックには互換する
* インテンション: 
  * インテンション排他ロック: インテンションロック同士では互換するが、排他と共有ロックには競合する
  * インテンション共有ロック: インテンションロック同士では互換するが、排他ロックには競合し、共有ロックには互換する

インテンションロックはテーブルロックに使われるが、こちらはあまり意識することはないかも。普段意識するのは、 INSERT 時にインテンションギャップロックが取得される、ということのほう。

* インテンションギャップロック同士は競合しない（同じインデックスギャップに挿入する複数のトランザクションは、ギャップ内のまったく同じ位置に挿入しない限りは競合しない）
* ギャップロック同士も同様に競合しない
* しかしギャップロックとインテンションギャップロックは競合する


それぞれの競合互換関係を以下表に整理する。

|                 | 排他 | インテンション排他 | 共有 | インテンション共有 | 
| --------------- | ---- | ---- | ---- | ---- |
| 排他             | 競合 | 競合 | 競合 | 競合 |
| インテンション排他 | 競合 | 互換 | 競合 | 互換 |
| 共有             | 競合 | 競合 | 互換 | 互換 |
| インテンション共有 | 競合 | 互換 | 互換 | 互換 |




## REPEATABLE READ におけるロックの範囲

REPEATABLE READ では、ファントムリードの発生を防止するために、インデックスレコードそのものへのロックだけでなく、インデックスレコード間のギャップやネクストキーへのロックが取得される場合があることに注意する。

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

```sql
SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 FOR UPDATE
```

以下のロックを取得する

* 7782 のレコードロック
* 7788 へのネクストキーロック、7839 へのネクストキーロック 

インデックスツリーのリーフノードを走査し、一つ先の 7839 までのロックが取得されることに注意する。

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
        L-----L-----L
```

### プライマリインデックスに対する範囲検索 + 追加条件

```sql
SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 AND ename LIKE '%t' FOR UPDATE
```

ロックの範囲は先述のプライマリインデックスに対する範囲検索と同様となる（結局インデックスツリーの走査なので）。
追加条件が一致するしないに関わらず同じになる。

```
(empno)

7698     7782    7788    7839
(blake)  (clark) (scott) (king)
  |-------|-------|-------| 
          L-------L-------L
```

### プライマリインデックスに対する等価検索 => 空振り

```sql
SELECT * FROM emp WHERE empno = 7785 FOR UPDATE
```

以下のロックを取得する。
空振りするとレコードロックではなくギャップロックとなり、ロックする範囲が広くなることに注意する。

* 7782 と 7788 の間のギャップロック

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
         L---L
```

### プライマリインデックスに対する範囲検索 => 空振り

```sql
SELECT * FROM emp WHERE empno BETWEEN 7784 AND 7786 FOR UPDATE
```

以下のロックを取得する。

* 7788 に対するネクストキーロック
* 7782 は含まれず、7782 から次のインデックスレコードまでのギャップを取得し、7788 にたどり着いて止まるイメージ

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
         L----L
```

### 非ユニークインデックスに対する範囲検索

```sql
SELECT * FROM emp WHERE job BETWEEN 'analyst' AND 'manager' FOR UPDATE
```

セカンダリインデックスに対し以下のロックが取得される。

* analyst (7788)、manager (7698、7782)、president (7839) に対するネクストキーロック

```
(job)

analyst  manager  manager  president       
(7788)   (7698)   (7782)   (7839)
   |--------|--------|--------| 
---L--------L--------L--------L
```

その後、プライマリインデックスに対し、以下のロックが取得される。

* 7698、7782、7788 に対するレコードロック

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
  L     L     L
```

### 非ユニークインデックスに対する等価条件

```sql
SELECT * FROM emp WHERE job = 'manager' FOR UPDATE
```

セカンダリインデックスに対し以下のロックが取得される。

* manager (7698、7782) に対するネクストキーロック
* president (7839) の手前までのギャップロック

```
(job)

analyst  manager  manager  president       
(7788)   (7698)   (7782)   (7839)
   |--------|--------|--------| 
    L-------L--------L-------L
```

その後、プライマリインデックスに対し、以下のロックが取得される。

* 7698、7782 に対するレコードロック

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
  L     L
```

非ユニークインデックスの場合は、等価条件であってもプライマリインデックスのときの範囲条件に近いロック範囲となることに注意する。

### 非ユニークインデックスに対する等価条件、フルスキャン

```sql
SELECT * FROM emp IGNORE INDEX (emp_job) WHERE job = 'manager' FOR UPDATE
```

以下のロックが取得される。

* 7698、7782、7788、7839、supremum に対するネクストキーロック
* supremum とは、内部的に設けられている上限値のレコードのこと

```
(empno)

7698   7782  7788  7839 supremum
  |-----|-----|-----|-----| 
--L-----L-----L-----L-----L
```

## READ COMMITTED における検索範囲

READ COMMITTED では、ファントムリードを許容するため、インデックスレコード間のギャップをロックする必要はなく、常にレコードロックとなることに注意する。また走査を行ったが検索条件に一致しなかったインデックスレコードのロックも SQL 文実行後に解放されるのもポイント

### プライマリインデックスに対する範囲検索

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 FOR UPDATE
```

以下のようにロックを取得する。

* インデックスツリーを走査し、7782、7788、7839 に対するレコードロックを取得する
* また SQL 実行後に、条件が一致しなかった 7839 のロックは解放される

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
        L     L    (L)
```

### プライマリインデックスに対する範囲検索＋追加条件

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
SELECT * FROM emp WHERE empno BETWEEN 7782 AND 7788 AND ename LIKE '%t' FOR UPDATE
```

以下のようにロックを取得する。

* インデックスツリーを走査し、7782、7788、7839 に対するレコードロックを取得する
* また SQL 実行後に、条件が一致しなかった 7782、7839 のロックは解放される

```
(empno)

7698     7782    7788    7839
(blake)  (clark) (scott) (king)
  |-------|-------|-------| 
         (L)      L      (L)
```


### プライマリインデックスに対する等価検索や範囲検索の空振り

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
SELECT * FROM emp WHERE empno = 7785 FOR UPDATE
```

ロックは取得されない。

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
```

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
SELECT * FROM emp WHERE empno BETWEEN 7784 AND 7786 FOR UPDATE
```

こちらもロックは取得されない。

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
```

## デッドロックするクエリの例

### 空振りするDELETE => INSERT

DELETE => INSERT するような処理で、DELETE が空振りする可能性がある場合、タイミングによりデッドロックが発生することがある。

```sql
TX1: REPEATABLE_READ
TX2: REPEATABLE_READ

TX1: DELETE FROM emp WHERE empno = 7784
TX2: DELETE FROM emp WHERE empno = 7786

TX1: INSERT INTO emp (empno, ename) VALUES (7784, 'steve')
TX2: INSERT INTO emp (empno, ename) VALUES (7786, 'bill')
```

最初の TX1 と TX2 による DELETE 文は空振りとなるため、以下のギャップロックを取得する。ギャップロック同士は競合しないことに注意。

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
TX1      L---L
TX2      L---L
```

次の TX1 による INSERT 文でインテンションギャップロックを取得しようとするが、TX2 が保持するギャップロックと競合するため待たされる。その後 TX2 が INSERT 文でインテンションギャップロックを取得しようとするが、TX1 が保持するするギャップロックと競合するため待たされデッドロックとなる。

```
(empno)

7698   7782  7788  7839
  |-----|-----|-----| 
TX1      L---L
TX2      L---L
TX1     (L---L) TX2 待ち
TX2     (L---L) TX1 待ち => デッドロック
```

回避策としては、DELETE の前に SELECT して、存在しているならば削除するというようにすれば、空振りによるギャップロック取得を防止することができ、デッドロックを回避できる。（SELECT はロックではなく MVCC のため、ロックと競合しない）

## appendix

### InnoDB ロックモニタ

InnoDB ロックモニタを有効にすると、`SHOW ENGINE INNODB STATUS` でトランザクションが保持しているロック情報を参照できる。

```sql
SET GLOBAL innodb_status_output_locks = ON
SHOW ENGINE INNODB STATUS
```
