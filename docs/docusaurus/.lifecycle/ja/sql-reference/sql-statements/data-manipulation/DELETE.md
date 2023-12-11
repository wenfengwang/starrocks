---
displayed_sidebar: "Japanese"
---

# DELETE

指定された条件に基づいてテーブルからデータ行を削除します。テーブルはパーティションされたテーブルまたは非パーティションされたテーブルになります。

Duplicate Keyテーブル、Aggregateテーブル、およびUnique Keyテーブルでは、指定されたパーティションからデータを削除できます。v2.3以降、Primary Keyテーブルは主キー、任意の列、またはサブクエリの結果に基づいてデータ行を削除する完全な「DELETE...WHERE」セマンティクスをサポートします。v3.0以降、StarRocksはマルチテーブル結合と共通テーブル式（CTE）を使用した「DELETE...WHERE」セマンティクスを拡張します。データベース内の他のテーブルとPrimary Keyテーブルを結合する必要がある場合は、USING句またはCTEでこれらの他のテーブルを参照できます。

## 使用上の注意

- DELETEを実行するためには、テーブルとデータベースに対する特権が必要です。
- 頻繁なDELETE操作は推奨されません。必要な場合は、オフピーク時にそのような操作を実行してください。
- DELETE操作はテーブル内のデータだけを削除します。テーブル自体は残ります。テーブルを削除するには、[DROP TABLE](../data-definition/DROP_TABLE.md)を実行してください。
- テーブル全体のデータを誤って削除するミス操作を防ぐために、DELETE文でWHERE句を指定する必要があります。
- 削除された行はすぐにはクリーンアップされません。それらは「削除済み」としてマークされ、一時的にセグメントに保存されます。物理的には、データバージョンのマージ（コンパクション）が完了した後に行が削除されます。
- この操作は、このテーブルを参照するマテリアライズドビューのデータも削除します。

## Duplicate Keyテーブル、Aggregateテーブル、およびUnique Keyテーブル

### 構文

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### パラメータ

| **パラメータ**   | **必須** | **説明**                                                     |
| :--------------- | :------- | :----------------------------------------------------------- |
| `db_name`        | いいえ   | 対象テーブルが属するデータベース。このパラメータを指定しない場合は、デフォルトで現在のデータベースが使用されます。      |
| `table_name`     | はい     | データを削除したいテーブル。   |
| `partition_name` | いいえ   | データを削除したいパーティション。  |
| `column_name`    | はい     | DELETEの条件として使用したい列。1つ以上の列を指定できます。   |
| `op`             | はい     | DELETEの条件で使用する演算子。サポートされる演算子は`=`、`>`、`<`、`>=`、`<=`、`!=`、`IN`、および`NOT IN`です。 |

### 制限および使用上の注意

- Duplicate Keyテーブルでは、**どの列**をDELETE条件として使用できます。AggregateテーブルとUnique Keyテーブルでは、DELETE条件として**主キー列**のみを使用できます。

- 指定する条件はAND関係である必要があります。OR関係で条件を指定したい場合は、個々のDELETE文で条件を指定する必要があります。

- Duplicate Keyテーブル、Aggregateテーブル、およびUnique KeyテーブルのDELETE文は、サブクエリの結果を条件として使用することはサポートしていません。

### 影響

DELETE文を実行すると、一定期間（コンパクションが完了するまで）クラスターのクエリパフォーマンスが低下する場合があります。低下の程度は指定した条件の数によって異なります。条件の数が多いほど、低下の程度も高くなります。

### 例

#### テーブルを作成しデータを挿入

次の例では、パーティションされたDuplicate Keyテーブルを作成します。

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

#### データをクエリ

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

#### データを削除

##### 指定されたパーティションからデータを削除

`p1`パーティションから `k1` の値が `3` の行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- クエリ結果に`k1`の値が`3`である行が削除されたことが表示されます。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### ANDを使用して指定されたパーティションからデータを削除

`p1`パーティションから `k1` の値が`3`以上かつ `k2` の値が`"abc"`の行を削除します。

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

##### 全てのパーティションからデータを削除

全てのパーティションから `k2` の値が`"abc"`または`"cba"`の行を削除します。

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

## Primary Keyテーブル

v2.3以降、Primary Keyテーブルは主キー、任意の列、またはサブクエリに基づいてデータ行を削除する完全な「DELETE...WHERE」セマンティクスをサポートします。

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
|   `with_query`    | いいえ    | DELETE文で名前で参照できる1つ以上のCTE。CTEは、複雑な文の可読性を向上させることができる一時的な結果セットです。   |
|   `table_name`    | はい      | データを削除したいテーブル。   |
|    `from_item`    | いいえ    | データベース内の1つ以上の他のテーブル。これらのテーブルは、WHERE句で指定された条件に基づいて、操作されるテーブルと結合できます。結合クエリの結果セットに応じて、StarRocksは操作されるテーブルから一致する行を削除します。例えば、USING句が`USING t1 WHERE t0.pk = t1.pk;`の場合、StarRocksはDELETE文を実行する際に、USING句のテーブル式を`t0 JOIN t1 ON t0.pk=t1.pk;`に変換します。
| `where_condition` | はい      | 行を削除する条件。WHERE条件を満たす行のみが削除されます。このパラメータが必要な理由は、誤ってテーブル全体を削除するのを防ぐためです。テーブル全体を削除したい場合は、「WHERE true」を使用します。 |

### 制限および使用上の注意

- Primary Keyテーブルでは、例えば `DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>` のように、指定されたパーティションからデータを削除することはサポートされていません。
- 次の比較演算子がサポートされています: `=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN`。
- 次の論理演算子がサポートされています: `AND` および `OR`。
- 以下のニーズに関連するデータの削除にDELETEステートメントを使用することはできません：同時DELETE操作の実行、データのロード時のデータの削除。このような操作を実行すると、トランザクションの原子性、整合性、分離性、耐久性（ACID）が保証されない可能性があります。

### 例

#### テーブルの作成とデータの挿入

`score_board`という名前のPrimary Keyテーブルを作成してください：

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

以下のステートメントを実行して`score_board`テーブルにデータを挿入してください：

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

##### 主キーによるデータの削除

DELETEステートメントで主キーを指定できるため、StarRocksはテーブル全体をスキャンする必要がありません。

`score_board`テーブルから`id`の値が`0`の行を削除してください。

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

例1：`score`の値が`22`の行を`score_board`テーブルから削除してください。

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

例2：`score`の値が`22`未満の行を`score_board`テーブルから削除してください。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

例3：`score`の値が`22`未満でかつ`name`の値が`Bob`でない行を`score_board`テーブルから削除してください。

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

##### サブクエリ結果によるデータの削除

DELETEステートメントの中で1つまたは複数のサブクエリをネストし、サブクエリの結果を条件として使用できます。

データを削除する前に、以下のステートメントを実行して`users`という別のテーブルを作成してください：

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

`users`テーブルにデータを挿入してください：

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

`users`テーブルから`country`の値が`China`である行を検索するサブクエリをネストし、その結果と同じ`name`の値を持つ行を`score_board`テーブルから削除してください。以下のいずれかの方法を使用して目的を達成できます：

- Method 1

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

- Method 2

```plain
DELETE FROM score_board
WHERE EXISTS (select name from users
              where score_board.name = users.name and country = "China");
    
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
2 rows in set (0.00 sec)
```

##### 複数テーブルの結合またはCTEを使用したデータの削除

"foo"というプロデューサーによって製作されたすべての映画を削除するには、以下のステートメントを実行してください：

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

上記のステートメントを読みやすくするために、CTEを使用して再度記述することもできます。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```