---
displayed_sidebar: "Japanese"
---

# unnest

## 説明

UNNESTは、配列を受け取り、その配列内の要素をテーブルの複数の行に変換するテーブル関数です。この変換は「フラット化」とも呼ばれます。

UNNESTを使用すると、STRING、ARRAY、BITMAPなどの一般的な変換を実装するためにLateral Joinを使用できます。詳細については、[Lateral join](../../../using_starrocks/Lateral_join.md)を参照してください。

v2.5 以降、UNNESTは可変長の配列パラメータを受け取ることができます。配列のタイプや長さ（要素の数）はさまざまです。配列の長さが異なる場合、最も大きな長さが優先され、これにより、この長さ未満の配列にはヌルが追加されます。詳細については、[Example 2](#example-2-unnest-takes-multiple-parameters)を参照してください。

## 構文

```Haskell
unnest(array0[, array1 ...])
```

## パラメータ

`array`: 変換したい配列。配列またはARRAYデータ型に評価できる式でなければなりません。1つ以上の配列または配列式を指定できます。

## 戻り値

配列から変換された複数の行を返します。戻り値のタイプは、配列内の要素の種類によって異なります。

配列でサポートされている要素のタイプについては、[ARRAY](../../sql-statements/data-types/Array.md)を参照してください。

## 使用上の注意

- UNNESTはテーブル関数です。Lateral Joinと共に使用する必要がありますが、キーワードのLateral Joinは明示的に指定する必要はありません。
- 配列式がNULLに評価されるか、空である場合、行は返されません。
- 配列内の要素がNULLの場合、その要素にはNULLが返されます。

## 例

### 例 1：UNNESTが1つのパラメータを取る場合

```SQL
-- scoresがARRAY列であるstudent_scoreテーブルを作成します。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- このテーブルにデータを挿入します。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- このテーブルからデータをクエリします。
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

-- UNNESTを使用して、scores列を複数の行に展開します。
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

[80,85,87]に対応する `id = 1` が3つの行に変換されます。

[77,null,89]に対応する `id = 2` はヌル値を保持します。

`id = 3` および `id = 4` に対応する`scores`はNULLおよび空なので、それらはスキップされます。

### 例 2：UNNESTが複数のパラメータを取る場合

```SQL
-- typeおよびscores列のタイプが異なるexample_tableテーブルを作成します。
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

-- このテーブルにデータを挿入します。
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- このテーブルからデータをクエリします。
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- UNNESTを使用して、typeおよびscoresを複数の行に変換します。
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

`UNNEST` の `type` と `scores` はタイプと長さが異なります。

`type`はVARCHAR列で、`scores`はARRAY列です。`split()` 関数を使用して、`type`をARRAYに変換します。

`id = 1` の場合、 `type` は["typeA","typeB"]に変換され、2つの要素が含まれます。

`id = 2` の場合、 `type` は["typeA","typeB","typeC"]に変換され、3つの要素が含まれます。

各 `id` に対して一貫した行数を確保するために、["typeA","typeB"]にヌル要素が追加されます。