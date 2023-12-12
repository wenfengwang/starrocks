---
displayed_sidebar: "Japanese"
---

# ブルームフィルター索引

このトピックでは、ブルームフィルター索引の作成と変更方法、およびその動作について説明します。

ブルームフィルター索引は、テーブルのデータファイル内でのフィルタリングされたデータの可能な存在を検出するために使用されるスペース効率のデータ構造です。ブルームフィルター索引が、特定のデータファイル内にフィルタリングするデータがないことを検出すると、StarRocksはデータファイルのスキャンをスキップします。ブルームフィルター索引は、列（例：IDなど）の基数が比較的高い場合に応答時間を短縮することができます。

クエリがソートキーカラムにヒットする場合、StarRocksは[プレフィックスインデックス](../table_design/Sort_key.md)を使用してクエリ結果を効率的に返します。ただし、データブロックのプレフィックスインデックスエントリは36バイトを超えることはできません。ソートキーとして使用されない列でクエリのパフォーマンスを向上させたい場合は、その列に対してブルームフィルター索引を作成できます。

## 動作方法

例えば、指定されたテーブル`table1`の`column1`にブルームフィルター索引を作成し、`Select xxx from table1 where column1 = something;`のようなクエリを実行した場合、StarRocksは`table1`のデータファイルをスキャンする際に以下の状況が発生します。

- ブルームフィルター索引が、データファイルがフィルタリングするデータを含んでいないことを検出した場合、StarRocksはクエリのパフォーマンスを向上させるためにデータファイルのスキャンをスキップします。
- ブルームフィルター索引が、データファイルがフィルタリングするデータを含んでいる可能性があることを検出した場合、StarRocksはデータファイルを読み取り、データの存在を確認します。なお、ブルームフィルターは値が存在しないことを確実に教えてくれますが、値が存在することを確実に言うことはできず、存在する可能性だけを伝えることができます。ブルームフィルター索引を使用して値が存在するかどうかを判断すると、偽の陽性が発生する可能性があります。つまり、ブルームフィルター索引がデータファイルがフィルタリングするデータを含んでいることを検出した場合でも、実際にはデータファイルにそのデータが含まれていない場合があります。

## 使用上の注意

- ユニークキーやプライマリーキーのテーブルには、すべての列に対してブルームフィルター索引を作成できます。集計テーブルやユニークキーテーブルの場合は、キーカラムに対してのみブルームフィルター索引を作成できます。
- TINYINT、FLOAT、DOUBLE、DECIMALの列はブルームフィルター索引を作成できません。
- ブルームフィルター索引は、`in` および `=` 演算子を含むクエリのパフォーマンスを向上させることができます（例： `Select xxx from table where x in {}` や `Select xxx from table where column = xxx`）。
- クエリがブルームフィルター索引を使用しているかどうかは、クエリのプロファイルの `BloomFilterFilterRows` フィールドを表示することで確認できます。

## ブルームフィルター索引の作成

テーブルを作成する際に、`PROPERTIES`内で`bloom_filter_columns`パラメータを指定することで列にブルームフィルター索引を作成できます。例えば、`table1`の`k1`と`k2`列にブルームフィルター索引を作成します。

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

これらの列名を指定して複数の列に対してブルームフィルター索引を一度に作成できます。ただし、これらの列名はコンマ（`,`）で区切る必要があります。`CREATE TABLE`ステートメントの他のパラメータの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

## ブルームフィルター索引の表示

例えば、次のステートメントは`table1`のブルームフィルター索引を表示します。出力の説明については、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を参照してください。

```SQL
SHOW CREATE TABLE table1;
```

## ブルームフィルター索引の変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)ステートメントを使用して、ブルームフィルター索引を追加、削減、および削除することができます。

- 次のステートメントは`v1`列にブルームフィルター索引を追加します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 次のステートメントは`k2`列のブルームフィルター索引を削減します。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 次のステートメントは`table1`のすべてのブルームフィルター索引を削除します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意：インデックスの変更は非同期操作です。この操作の進行状況は、[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)を実行して確認できます。テーブルごとに1回のインデックス変更タスクしか実行できません。