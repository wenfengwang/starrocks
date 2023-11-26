---
displayed_sidebar: "Japanese"
---

# ブルームフィルターインデックス

このトピックでは、ブルームフィルターインデックスの作成と変更方法について説明します。

ブルームフィルターインデックスは、テーブルのデータファイルにおけるフィルタリングされたデータの可能な存在を検出するために使用される、スペース効率の高いデータ構造です。ブルームフィルターインデックスは、データファイルにフィルタリングするデータが含まれていないことを検出した場合、StarRocksはデータファイルのスキャンをスキップします。ブルームフィルターインデックスは、列（例：ID）の基数が比較的高い場合に応答時間を短縮することができます。

クエリがソートキーカラムにヒットする場合、StarRocksは[プレフィックスインデックス](../table_design/Sort_key.md)を使用してクエリ結果を効率的に返します。ただし、データブロックのプレフィックスインデックスエントリの長さは36バイトを超えることはできません。ソートキーとして使用されず、基数が比較的高い列のクエリパフォーマンスを向上させたい場合は、その列に対してブルームフィルターインデックスを作成することができます。

## 動作原理

例えば、与えられたテーブル`table1`の`column1`に対してブルームフィルターインデックスを作成し、`Select xxx from table1 where column1 = something;`というクエリを実行すると、StarRocksが`table1`のデータファイルをスキャンする際に以下の状況が発生します。

- ブルームフィルターインデックスがフィルタリングするデータが含まれていないデータファイルを検出した場合、StarRocksはクエリパフォーマンスを向上させるためにデータファイルのスキャンをスキップします。
- ブルームフィルターインデックスがフィルタリングするデータが含まれている可能性のあるデータファイルを検出した場合、StarRocksはデータファイルを読み込んでデータが存在するかどうかを確認します。なお、ブルームフィルターは値が存在しないことを確実に伝えることができますが、値が存在することを確実に伝えることはできません。ブルームフィルターインデックスを使用して値の存在を判断すると、誤検出（false positives）が発生する可能性があります。つまり、ブルームフィルターインデックスがデータファイルがフィルタリングするデータを含んでいると検出するが、実際にはデータファイルはそのデータを含んでいない場合です。

## 使用上の注意

- 重複キーやプライマリキーのテーブルのすべての列に対してブルームフィルターインデックスを作成することができます。集計テーブルやユニークキーのテーブルの場合、キーカラムに対してのみブルームフィルターインデックスを作成することができます。
- TINYINT、FLOAT、DOUBLE、DECIMALの列はブルームフィルターインデックスの作成をサポートしていません。
- ブルームフィルターインデックスは、`in`および`=`演算子を含むクエリのパフォーマンスを向上させることができます。例えば、`Select xxx from table where x in {}`や`Select xxx from table where column = xxx`などです。
- クエリがブルームフィルターインデックスを使用しているかどうかは、クエリのプロファイルの`BloomFilterFilterRows`フィールドを表示することで確認できます。

## ブルームフィルターインデックスの作成

`PROPERTIES`で`bloom_filter_columns`パラメータを指定してテーブルを作成する際に、列に対してブルームフィルターインデックスを作成することができます。例えば、`table1`の`k1`と`k2`の列に対してブルームフィルターインデックスを作成します。

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

これらの列名を指定して、複数の列に対して一度にブルームフィルターインデックスを作成することができます。ただし、これらの列名をカンマ（`,`）で区切る必要があります。CREATE TABLE文の他のパラメータの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

## ブルームフィルターインデックスの表示

例えば、次の文は`table1`のブルームフィルターインデックスを表示します。出力の説明については、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を参照してください。

```SQL
SHOW CREATE TABLE table1;
```

## ブルームフィルターインデックスの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)文を使用して、ブルームフィルターインデックスを追加、削減、削除することができます。

- 次の文は`v1`列にブルームフィルターインデックスを追加します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 次の文は`k2`列のブルームフィルターインデックスを削減します。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 次の文は`table1`のすべてのブルームフィルターインデックスを削除します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意: インデックスの変更は非同期操作です。[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)を実行することで、この操作の進行状況を表示することができます。テーブルごとに一度に1つのインデックス変更タスクのみ実行できます。
