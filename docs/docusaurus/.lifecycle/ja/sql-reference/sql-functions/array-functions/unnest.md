---
displayed_sidebar: "Japanese"
---

# unnest

## 説明

UNNESTは、配列を取り、その配列の要素をテーブルの複数の行に変換するテーブル関数です。この変換は"フラット化"としても知られています。

UNNESTを使用して共通の変換（例: STRING、ARRAY、またはBITMAPから複数の行への変換）を実装するために、UNNESTとLateral Joinを使用することができます。詳細については、[Lateral join](../../../using_starrocks/Lateral_join.md)を参照してください。

v2.5から、UNNESTは可変長の配列パラメーターを取ることができます。配列は種類や要素の数（長さ）が異なることができます。配列の長さが異なる場合、最大の長さが優先され、つまりその長さよりも短い配列にはnullが追加されます。詳細については、[Example 2](#example-2-unnest-takes-multiple-parameters)を参照してください。

## 構文

```Haskell
unnest(array0[, array1 ...])
```

## パラメーター

`array`: 変換したい配列。これは、配列またはARRAYデータ型に評価できる式でなければなりません。1つまたは複数の配列または配列式を指定できます。

## 戻り値

配列から変換された複数の行が返されます。戻り値の型は、配列の要素の型によって異なります。

配列内でサポートされている要素の種類については、[ARRAY](../../sql-statements/data-types/Array.md)を参照してください。

## 使用上の注意

- UNNESTはテーブル関数です。Lateral Joinと共に使用する必要がありますが、キーワードのLateral Joinを明示的に指定する必要はありません。
- 配列式がNULLに評価されるか空である場合、行が返されません。
- 配列内の要素がNULLの場合、その要素にはNULLが返されます。

## 例

### 例1: UNNESTは1つのパラメーターを取る

```SQL
-- scoresがARRAYカラムであるstudent_scoreテーブルを作成する。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- このテーブルにデータを挿入する。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- このテーブルからデータをクエリする。
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

-- UNNESTを使用してscoresカラムを複数の行に展開する。
SELECT id, scores, unnest FROM student_score, unnest(scores);
+------+--------------+--------+
| id   | scores       | unnest |
+------+--------------+--------+
|    1 | [80,85,87]   |     80 |
|    1 | [80,85,87]   |     85 |
|    1 | [80,85,87]   |     87 |
|    2 | [77,null,89]  |     77 |
|    2 | [77,null,89]  |   NULL |
|    2 | [77,null,89]  |     89 |
|    5 | [90,92]      |     90 |
|    5 | [90,92]      |     92 |
+------+--------------+--------+
```

[80,85,87]は`id = 1`に対応して3つの行に変換されます。

[77,null,89]は`id = 2`に対応してnull値を保持します。

`id = 3`および`id = 4`の`scores`はNULLおよび空のため、スキップされます。

### 例2: UNNESTは複数のパラメーターを取る

```SQL
-- typeとscoresの種類が異なるexample_tableを作成する。
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

-- このテーブルにデータを挿入する。
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- このテーブルからデータをクエリする。
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- UNNESTを使用してtypeとscoresを複数の行に変換する。
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

`UNNEST`の`type`および`scores`は種類と長さが異なります。

`type`はVARCHARカラムで、`scores`はARRAYカラムです。`split()`関数は`type`をARRAYに変換するために使用されます。

`id = 1`の場合、`type`は["typeA", "typeB"]に変換され、2つの要素を持ちます。

`id = 2`の場合、`type`は["typeA", "typeB", "typeC"]に変換され、3つの要素を持ちます。

各`id`について一貫した行数を確保するために、["typeA", "typeB"]にはnull要素が追加されます。