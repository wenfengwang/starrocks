---
displayed_sidebar: Chinese
---

# Bloom filter 索引

本文介绍了 Bloom filter（ブルームフィルター）索引の原理と、Bloom filter 索引の作成や変更方法について説明します。

Bloom filter 索引は、テーブルのデータファイルに検索対象のデータが含まれている可能性を迅速に判断し、含まれていない場合はスキップしてデータのスキャン量を減らすことができます。Bloom filter 索引は空間効率が高く、基数が高い列（例えば ID 列）に適しています。クエリ条件がプレフィックスインデックスの列にヒットした場合、StarRocks は[プレフィックスインデックス](../table_design/Sort_key.md)を使用してクエリ結果を迅速に返します。しかし、プレフィックスインデックスの長さには限りがあり、非プレフィックスインデックスの列で基数が高い列を迅速に検索したい場合は、その列に Bloom filter 索引を作成することができます。

## 索引原理

例えば、`table1` の `column1` 列に Bloom filter 索引を作成し、SQL クエリ `Select xxx from table1 where column1 = something;` がインデックスにヒットした場合、データファイルのスキャン時には次の2つの状況が発生します：

- Bloom filter 索引がデータファイルに目的のデータが存在しないと判断した場合、StarRocks はそのファイルをスキップし、クエリの効率を向上させます。
- Bloom filter 索引がデータファイルに目的のデータが存在する可能性があると判断した場合、StarRocks はそのファイルを読み込んで目的のデータが実際に存在するかを確認します。ただし、これはファイルに目的のデータが含まれる可能性があるという判断に過ぎません。Bloom filter 索引には一定の誤判率、つまり偽陽性率（False Positive Probability）があり、目的のデータが含まれていると判断されるが、実際には含まれていない可能性があります。

## 使用説明

- 主キーモデルと明細モデルのすべての列に Bloom filter 索引を作成できます；集約モデルと更新モデルでは、次元列（Key 列）のみが Bloom filter 索引の作成をサポートしています。
- TINYINT、FLOAT、DOUBLE、DECIMAL 型の列には Bloom filter 索引を作成することはできません。
- Bloom filter 索引は `in` や `=` のフィルター条件を含むクエリの効率を向上させることができます。例えば `Select xxx from table where xxx in ()` や `Select xxx from table where column = xxx` のようなクエリです。
- クエリが Bloom filter 索引にヒットしたかどうかを知るには、クエリの Profile の `BloomFilterFilterRows` フィールドを確認してください。Profile の確認方法については、[クエリの分析](../administration/Query_planning.md#プロファイルの確認)を参照してください。

## インデックスの作成

テーブル作成時に `PROPERTIES` で `bloom_filter_columns` を指定することでインデックスを作成できます。例えば、以下のステートメントは `table1` の `k1` と `k2` 列に Bloom filter 索引を作成します。

```SQL
CREATE TABLE table1
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT DEFAULT "10"
)
ENGINE = olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES("bloom_filter_columns" = "k1,k2");
```

複数のインデックスを同時に作成することができ、インデックス列はコンマ（`,`）で区切ります。テーブル作成の他のパラメータについては、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

## インデックスの確認

例えば、`table1` のインデックスを確認します。戻り値の説明については、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) を参照してください。

```SQL
SHOW CREATE TABLE table1;
```

## インデックスの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して、インデックスを追加、減少、削除することができます。

- 以下のステートメントは、Bloom filter 索引列 `v1` を追加しました。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 以下のステートメントは、Bloom filter 索引列 `k2` を減少しました。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 以下のステートメントは、`table1` のすべての Bloom filter 索引を削除しました。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意：インデックスの変更は非同期操作であり、[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md) コマンドを使用してインデックス変更の進行状況を確認できます。現在、各テーブルは同時に1つのインデックス変更タスクのみを許可しています。
