---
displayed_sidebar: "Japanese"
---

# ブルームフィルターインデックス

このトピックでは、ブルームフィルターインデックスの作成および変更方法、およびその動作について説明します。

ブルームフィルターインデックスは、テーブルのデータファイル内でフィルタリングされたデータの可能性の存在を検出するために使用されるデータ構造であり、データのスキャンをスキップすることでレスポンス時間を短縮できます。ブルームフィルターインデックスは、列（たとえばIDなど）の基数が比較的高い場合にレスポンス時間を短縮できます。

クエリがソートキー列にヒットする場合、StarRocksは[プレフィックスインデックス](../table_design/Sort_key.md)を利用してクエリ結果を効率的に返します。ただし、データブロックのプレフィックスインデックスのエントリは36バイトを超えてはなりません。したがって、ソートキーとして使用されておらず、基数が比較的高い列に対するクエリパフォーマンスを改善するには、その列に対してブルームフィルターインデックスを作成できます。

## 動作方法

たとえば、任意のテーブル`table1`の`column1`にブルームフィルターインデックスを作成し、`Select xxx from table1 where column1 = something;`というクエリを実行した場合、StarRocksが`table1`のデータファイルをスキャンする際に、以下のような状況が発生します。

- ブルームフィルターインデックスがデータファイルにフィルタリングされたデータが含まれていないことを検出した場合、StarRocksはデータファイルのスキャンをスキップしてクエリのパフォーマンスを向上させます。
- ブルームフィルターインデックスがデータファイルにフィルタリングされたデータが含まれる可能性があることを検出した場合、StarRocksはデータファイルを読み取り、データの存在を確認します。なお、ブルームフィルターは値が存在しないことを確実に伝えることができますが、値が存在することを確実に伝えることはできず、存在する可能性のみを示すことができます。ブルームフィルターインデックスを使用して値の存在を判断すると、誤検知が発生する可能性があります。つまり、ブルームフィルターインデックスがデータファイルにフィルタリングされたデータが含まれると検出するが、実際には含まれていない状況が発生する可能性があります。

## 使用上の注意

- Duplicate KeyテーブルまたはPrimary Keyテーブルのすべての列に対してブルームフィルターインデックスを作成できます。AggregateテーブルまたはUnique Keyテーブルでは、キーカラムに対してのみブルームフィルターインデックスを作成できます。
- TINYINT、FLOAT、DOUBLE、DECIMAL列はブルームフィルターインデックスの作成をサポートしていません。
- ブルームフィルターインデックスは、`in`および`=`オペレータを含むクエリのパフォーマンスを向上させることができます。例：`Select xxx from table where x in {}`および`Select xxx from table where column = xxx`。
- クエリがブルームフィルターインデックスを使用しているかどうかは、クエリのプロファイルの`BloomFilterFilterRows`フィールドを表示することで確認できます。

## ブルームフィルターインデックスの作成

`PROPERTIES`で`bloom_filter_columns`パラメータを指定してテーブルを作成する際に、列に対してブルームフィルターインデックスを作成できます。たとえば、`table1`の`k1`および`k2`列に対してブルームフィルターインデックスを作成します。

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

これらの列名を指定して複数の列に対して一度にブルームフィルターインデックスを作成できます。なお、これらの列名はコンマ（`,`）で区切る必要があります。`CREATE TABLE`ステートメントのその他のパラメータの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

## ブルームフィルターインデックスの表示

たとえば、次のステートメントでは`table1`のブルームフィルターインデックスを表示します。出力の詳細については、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を参照してください。

```SQL
SHOW CREATE TABLE table1;
```

## ブルームフィルターインデックスの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)ステートメントを使用して、ブルームフィルターインデックスを追加、減少、削除することができます。

- 次のステートメントは、`v1`列にブルームフィルターインデックスを追加します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 次のステートメントは、`k2`列のブルームフィルターインデックスを減少します。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 次のステートメントは、`table1`のすべてのブルームフィルターインデックスを削除します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注意：インデックスの変更は非同期操作です。この操作の進捗状況は[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)を実行して確認できます。テーブルごとに1つのインデックス操作のみを実行できます。