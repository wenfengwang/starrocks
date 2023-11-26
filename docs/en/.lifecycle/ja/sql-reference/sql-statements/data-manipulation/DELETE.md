---
displayed_sidebar: "Japanese"
---

# DELETE（削除）

指定された条件に基づいて、テーブルからデータ行を削除します。テーブルはパーティション化または非パーティション化のテーブルである場合があります。

重複キーのテーブル、集計テーブル、およびユニークキーのテーブルでは、指定したパーティションからデータを削除することができます。ただし、プライマリキーのテーブルでは、それはできません。v2.3以降、プライマリキーのテーブルは完全な `DELETE...WHERE` のセマンティクスをサポートし、プライマリキー、任意の列、またはサブクエリの結果に基づいてデータ行を削除することができます。v3.0以降、StarRocksは `DELETE...WHERE` のセマンティクスをマルチテーブルの結合と共通テーブル式（CTE）で拡張しています。データベース内の他のテーブルとプライマリキーのテーブルを結合する必要がある場合は、USING句またはCTEでこれらの他のテーブルを参照することができます。

## 使用上の注意

- 削除を実行するテーブルとデータベースには権限が必要です。
- 頻繁な削除操作は推奨されません。必要な場合は、オフピーク時に実行してください。
- 削除操作はテーブル内のデータのみを削除します。テーブル自体は残ります。テーブルを削除するには、[DROP TABLE](../data-definition/DROP_TABLE.md) を実行してください。
- テーブル全体のデータを誤って削除することを防ぐために、DELETE文のWHERE句を指定する必要があります。
- 削除された行はすぐにクリーンアップされません。それらは「削除済み」としてマークされ、一時的にセグメントに保存されます。物理的には、データバージョンのマージ（コンパクション）が完了した後に行が削除されます。
- この操作は、このテーブルを参照するマテリアライズドビューのデータも削除します。

## 重複キーのテーブル、集計テーブル、およびユニークキーのテーブル

### 構文

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### パラメータ

| **パラメータ**         | **必須** | **説明**                                                     |
| :--------------- | :------- | :----------------------------------------------------------- |
| `db_name`        | いいえ       | 対象のテーブルが所属するデータベースです。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。      |
| `table_name`     | はい      | データを削除するテーブルです。   |
| `partition_name` | いいえ       | データを削除するパーティションです。  |
| `column_name`    | はい      | 削除条件として使用する列です。1つまたは複数の列を指定できます。   |
| `op`             | はい      | 削除条件で使用する演算子です。サポートされている演算子は `=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN` です。 |

### 制限事項

- 重複キーのテーブルでは、削除条件として**任意の列**を使用することができます。集計テーブルとユニークキーのテーブルでは、削除条件として**キーカラム**のみを使用することができます。

- 指定する条件はANDの関係にする必要があります。条件をORの関係にする場合は、別々のDELETE文で条件を指定する必要があります。

- 重複キーのテーブル、集計テーブル、およびユニークキーのテーブルでは、DELETE文ではサブクエリの結果を条件として使用することはサポートされていません。

### 影響

DELETE文を実行した後、クラスタのクエリパフォーマンスが一時的に低下する場合があります（コンパクションが完了するまで）。低下の度合いは、指定する条件の数によって異なります。条件の数が多いほど、低下の度合いも高くなります。

### 例

#### テーブルの作成とデータの挿入

以下の例では、パーティション化された重複キーのテーブルを作成します。

```SQL
CREATE TABLE `my_table` (
    `date` date NOT NULL,
    `k1` int(11) NOT NULL COMMENT "",
    `k2` varchar(65533) NULL DEFAULT "" COMMENT "")
DUPLICATE KEY(`date`)
PARTITION BY RANGE(`date`)
(
    PARTITION p1 VALUES [('2022-03-11'), ('2022-03-12')),
    PARTITION p2 VALUES [('2022-03-12'), ('2022-03-13'))
)
DISTRIBUTED BY HASH(`date`)
PROPERTIES
("replication_num" = "3");

INSERT INTO `my_table` VALUES
('2022-03-11', 3, 'abc'),
('2022-03-11', 2, 'acb'),
('2022-03-11', 4, 'abc'),
('2022-03-12', 2, 'bca'),
('2022-03-12', 4, 'cba'),
('2022-03-12', 5, 'cba');
```

#### データのクエリ

以下のステートメントを実行して、`my_table` テーブルにデータを挿入します。

```plain
select * from my_table order by date;
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    3 | abc  |
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
| 2022-03-12 |    2 | bca  |
| 2022-03-12 |    4 | cba  |
| 2022-03-12 |    5 | cba  |
+------------+------+------+
```

#### データの削除

##### 指定したパーティションからデータを削除

`p1` パーティションから `k1` の値が `3` の行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- クエリの結果、`k1` の値が `3` の行が削除されていることが表示されます。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### AND を使用して指定したパーティションからデータを削除

`p1` パーティションから `k1` の値が `3` 以上で、かつ `k2` の値が `"abc"` の行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 >= 3 AND k2 = "abc";

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
+------------+------+------+
```

##### すべてのパーティションからデータを削除

すべてのパーティションから `k2` の値が `"abc"` または `"cba"` の行を削除します。

```plain
DELETE FROM my_table
WHERE  k2 in ("abc", "cba");

select * from my_table order by date;
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-12 |    2 | bca  |
+------------+------+------+
```

## プライマリキーのテーブル

v2.3以降、プライマリキーのテーブルは完全な `DELETE...WHERE` のセマンティクスをサポートし、プライマリキー、任意の列、またはサブクエリに基づいてデータ行を削除することができます。

### 構文

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### パラメータ

|   **パラメータ**   | **必須** | **説明**                                              |
| :---------------: | :----------- | :----------------------------------------------------------- |
|   `with_query`    | いいえ           | DELETE文で参照できる1つ以上のCTE（共通テーブル式）です。CTEは、複雑なステートメントの可読性を向上させるための一時的な結果セットです。 |
|   `table_name`    | はい          | データを削除するテーブルです。                |
|    `from_item`    | いいえ           | データベース内の他の1つ以上のテーブルです。これらのテーブルは、WHERE句で指定された条件に基づいて操作対象のテーブルと結合することができます。結合クエリの結果セットに基づいて、StarRocksは操作対象のテーブルから一致する行を削除します。たとえば、USING句が `USING t1 WHERE t0.pk = t1.pk;` の場合、StarRocksはDELETE文を実行する際に、USING句内のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk;` に変換します。 |
| `where_condition` | はい          | 行を削除する基準となる条件です。WHERE条件を満たす行のみが削除されます。このパラメータは必須です。なぜなら、テーブル全体を誤って削除することを防ぐために役立つからです。テーブル全体を削除する場合は、'WHERE true' を使用できます。 |

### 制限事項

- 次の比較演算子がサポートされています：`=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN`。

- 次の論理演算子がサポートされています：`AND` および `OR`。

- DELETE文を使用して同時に複数のDELETE操作を実行したり、データのロード時にデータを削除したりすることはできません。このような操作を行うと、トランザクションの原子性、一貫性、分離性、耐久性（ACID）が保証されない場合があります。

### 例

#### テーブルの作成とデータの挿入

`score_board` という名前のプライマリキーのテーブルを作成します。

```sql
CREATE TABLE `score_board` (
  `id` int(11) NOT NULL COMMENT "",
  `name` varchar(65533) NULL DEFAULT "" COMMENT "",
  `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
) ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "false"
);

INSERT INTO score_board VALUES
(0, 'Jack', 21),
(1, 'Bob', 21),
(2, 'Stan', 21),
(3, 'Sam', 22);
```

#### データのクエリ

以下のステートメントを実行して、`score_board` テーブルにデータを挿入します。

```Plain
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    0 | Jack |   21 |
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
4 rows in set (0.00 sec)
```

#### データの削除

##### プライマリキーによるデータの削除

DELETE文でプライマリキーを指定することができるため、StarRocksはテーブル全体をスキャンする必要はありません。

`score_board` テーブルから `id` の値が `0` の行を削除します。

```Plain
DELETE FROM score_board WHERE id = 0;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
```

##### 条件によるデータの削除

例1：`score` の値が `22` の行を `score_board` テーブルから削除します。

```Plain
DELETE FROM score_board WHERE score = 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    0 | Jack |   21 |
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
+------+------+------+
```

例2：`score` の値が `22` より小さい行を `score_board` テーブルから削除します。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

例3：`score` の値が `22` より小さく、かつ `name` の値が `"Bob"` でない行を `score_board` テーブルから削除します。

```Plain
DELETE FROM score_board WHERE score < 22 and name != "Bob";

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    1 | Bob  |   21 |
|    3 | Sam  |   22 |
+------+------+------+
2 rows in set (0.00 sec)
```

##### サブクエリの結果によるデータの削除

DELETE文内に1つ以上のサブクエリをネストし、サブクエリの結果を条件として使用することができます。

データを削除する前に、以下のステートメントを実行して `users` という別のテーブルを作成します。

```SQL
CREATE TABLE `users` (
  `uid` int(11) NOT NULL COMMENT "",
  `name` varchar(65533) NOT NULL COMMENT "",
  `country` varchar(65533) NULL COMMENT ""
) ENGINE=OLAP
PRIMARY KEY(`uid`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`uid`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "false"
);
```

`users` テーブルにデータを挿入します。

```SQL
INSERT INTO users VALUES
(0, "Jack", "China"),
(2, "Stan", "USA"),
(1, "Bob", "China"),
(3, "Sam", "USA");
```

```plain
select * from users;
+------+------+---------+
| uid  | name | country |
+------+------+---------+
|    0 | Jack | China   |
|    1 | Bob  | China   |
|    2 | Stan | USA     |
|    3 | Sam  | USA     |
+------+------+---------+
4 rows in set (0.00 sec)
```

`users` テーブルから `country` の値が `"China"` の行を返すサブクエリをネストし、その結果と同じ `name` の値を持つ行を `score_board` テーブルから削除します。目的を達成するために、次のいずれかの方法を使用できます。

- 方法1

```plain
DELETE FROM score_board
WHERE name IN (select name from users where country = "China");
    
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
```

- 方法2

```plain
DELETE FROM score_board
WHERE EXISTS (select name from users
              where score_board.name = users.name and country = "China");
    
select * from score_board;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|    2 | Stan |    21 |
|    3 | Sam  |    22 |
+------+------+-------+
2 rows in set (0.00 sec)
```

##### マルチテーブルの結合またはCTEを使用したデータの削除

プロデューサーが "foo" のすべての映画を削除するには、次のステートメントを実行します。

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

可読性を向上させるために、CTEを使用して上記のステートメントを書き直すこともできます。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```
