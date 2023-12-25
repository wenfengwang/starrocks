---
displayed_sidebar: Chinese
---

# unnest

## 機能

UNNEST はテーブル関数で、配列を複数の行に展開するために使用されます。

StarRocks の Lateral Join と UNNEST 機能を組み合わせて、STRING、ARRAY、BITMAP 型のデータなど、列から行への一般的な変換ロジックを実現することができます。使用例については、[Lateral join](../../../using_starrocks/Lateral_join.md) を参照してください。

バージョン 2.5 から、UNNEST は複数の array 引数を受け入れるようになり、それぞれの array の要素の型と長さ（要素数）が異なっても構いません。長さが異なる場合は、最長の配列の長さを基準にして、それより短い配列は NULL を使って要素を補完します。詳細は [例二](#例二unnest-複数の引数を受け取る) を参照してください。

## 文法

```Haskell
unnest(array0[, array1 ...])
```

## 引数説明

`array`：変換される配列、または配列に変換可能な式。必須です。

### 戻り値の説明

配列を展開した後の複数の行を返します。戻り値のデータ型は配列内の要素の型に依存します。

StarRocks がサポートする配列の要素の型については、[ARRAY](../../sql-statements/data-types/Array.md) を参照してください。

## 注意事項

- UNNEST は lateral join と共に使用する必要がありますが、クエリ内で lateral join キーワードを省略することができます。
- 複数の配列を入力としてサポートし、配列の長さと型が異なっても構いません。
- 入力された配列が NULL または空の場合は、計算時にスキップされます。
- 配列内の要素が NULL の場合、その要素に対応する位置は NULL を返します。

## **例**

### 例一：UNNEST が一つの引数を受け取る

```SQL
-- student_score テーブルを作成し、scores 列は ARRAY 型です。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- テーブルにデータを挿入します。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- テーブルのデータをクエリします。
SELECT * FROM student_score ORDER BY id;
+------+--------------+
| id   | scores       |
+------+--------------+
|    1 | [80,85,87]   |
|    2 | [77,null,89] |
|    3 | NULL         |
|    4 | []           |
|    5 | [90,92]      |
+------+--------------+

-- scores 列の配列要素を複数の行に展開します。
SELECT id, scores, unnest FROM student_score, unnest(scores);
+------+--------------+--------+
| id   | scores       | unnest |
+------+--------------+--------+
|    1 | [80,85,87]   |     80 |
|    1 | [80,85,87]   |     85 |
|    1 | [80,85,87]   |     87 |
|    2 | [77,null,89] |     77 |
|    2 | [77,null,89] |   NULL |
|    2 | [77,null,89] |     89 |
|    5 | [90,92]      |     90 |
|    5 | [90,92]      |     92 |
+------+--------------+--------+
```

`id = 1` の `scores` 配列は要素数に基づいて 3 行に分割されています。

`id = 2` の `scores` 配列には null 要素が含まれており、対応する位置では null が返されます。

`id = 3` と `id = 4` の `scores` 配列はそれぞれ NULL と空で、計算時にスキップされます。

### 例二：UNNEST が複数の引数を受け取る

```SQL
-- テーブルを作成します。
CREATE TABLE example_table (
id varchar(65533) NULL COMMENT "",
type varchar(65533) NULL COMMENT "",
scores ARRAY<int> NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(id)
COMMENT "OLAP"
DISTRIBUTED BY HASH(id)
PROPERTIES (
"replication_num" = "3");

-- テーブルにデータを挿入します。
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- テーブルのデータをクエリします。
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- UNNEST を使用して type と scores の両方の列の要素を複数の行に展開します。
SELECT id, unnest.type, unnest.scores
FROM example_table, unnest(split(type, ";"), scores) as unnest(type,scores);
+------+-------+--------+
| id   | type  | scores |
+------+-------+--------+
| 1    | typeA |     80 |
| 1    | typeB |     85 |
| 1    | NULL  |     88 |
| 2    | typeA |     87 |
| 2    | typeB |     90 |
| 2    | typeC |     95 |
+------+-------+--------+
```

`UNNEST` 関数の `type` 列と `scores` 列のデータ型は異なります。

`type` 列は VARCHAR 型で、計算プロセス中に split() 関数を使用して ARRAY 型に変換されます。

`id = 1` の `type` は ["typeA","typeB"] という配列に変換され、2 つの要素が含まれています。`id = 2` の `type` は ["typeA","typeB","typeC"] という配列に変換され、3 つの要素が含まれています。最長の配列の長さ 3 を基準にして、["typeA","typeB"] に NULL を補完しています。
