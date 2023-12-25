---
displayed_sidebar: English
---

# ブルームフィルターインデックスの作成

このトピックでは、ブルームフィルターインデックスの作成と変更方法、およびその動作原理について説明します。

ブルームフィルターインデックスは、テーブルのデータファイル内にフィルタリング対象データの存在可能性を検出するために使用される、空間効率の高いデータ構造です。ブルームフィルターインデックスがフィルタリング対象データが特定のデータファイルに含まれていないと検出した場合、StarRocksはそのデータファイルのスキャンをスキップします。ブルームフィルターインデックスは、カーディナリティが比較的高い列（例えばID）において、応答時間を短縮することができます。

クエリがソートキーカラムにヒットする場合、StarRocksは[prefix index](../table_design/Sort_key.md)を使用して効率的にクエリ結果を返します。しかし、データブロックのプレフィックスインデックスエントリの長さは36バイトを超えてはなりません。ソートキーとして使用されていないが、カーディナリティが比較的高いカラムのクエリパフォーマンスを向上させたい場合は、そのカラムにブルームフィルターインデックスを作成できます。

## 動作原理

例えば、あるテーブル`table1`の`column1`にブルームフィルターインデックスを作成し、`Select xxx from table1 where column1 = something;`というクエリを実行します。StarRocksが`table1`のデータファイルをスキャンする際、以下のような状況が発生します。

- ブルームフィルターインデックスがデータファイルにフィルタリング対象データが含まれていないと検出した場合、StarRocksはそのデータファイルのスキャンをスキップしてクエリパフォーマンスを向上させます。
- ブルームフィルターインデックスがデータファイルにフィルタリング対象データが含まれている可能性があると検出した場合、StarRocksはデータファイルを読み取り、データの存在を確認します。ブルームフィルターは、値が存在しないことは確実に伝えることができますが、値が存在することは確実には言えず、存在する可能性があることだけを示すことができる点に注意してください。ブルームフィルターインデックスを使用して値の存在を判断すると、誤検出が発生する可能性があります。つまり、ブルームフィルターインデックスがデータファイルにフィルタリング対象データが含まれていると検出しても、実際にはデータが含まれていない場合があります。

## 使用上の注意点

- ブルームフィルターインデックスは、Duplicate KeyテーブルまたはPrimary Keyテーブルの全てのカラムに対して作成できます。AggregateテーブルやUnique Keyテーブルでは、キーカラムにのみブルームフィルターインデックスを作成できます。
- TINYINT、FLOAT、DOUBLE、DECIMAL型のカラムはブルームフィルターインデックスの作成をサポートしていません。
- ブルームフィルターインデックスは、`in`や`=`の演算子を含むクエリのパフォーマンスのみを向上させることができます。例: `Select xxx from table where x in {}`や`Select xxx from table where column = xxx`。
- クエリがブルームフィルターインデックスを使用しているかどうかは、クエリのプロファイルにある`BloomFilterFilterRows`フィールドを確認することで判断できます。

## ブルームフィルターインデックスの作成

テーブルを作成する際に`PROPERTIES`内の`bloom_filter_columns`パラメータを指定することで、カラムにブルームフィルターインデックスを作成できます。例えば、`table1`の`k1`と`k2`カラムにブルームフィルターインデックスを作成する場合は以下のようにします。

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
DISTRIBUTED BY HASH(k1, k2)
PROPERTIES("bloom_filter_columns" = "k1,k2");
```

これらのカラム名を指定することで、一度に複数のカラムにブルームフィルターインデックスを作成できます。カラム名はコンマ(`,`)で区切る必要があります。CREATE TABLE文のその他のパラメータの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

## ブルームフィルターインデックスの表示

例えば、以下のステートメントは`table1`のブルームフィルターインデックスを表示します。出力の説明については、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を参照してください。

```SQL
SHOW CREATE TABLE table1;
```

## ブルームフィルターインデックスの変更

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)文を使用して、ブルームフィルターインデックスを追加、削減、または削除できます。

- 次のステートメントは、`v1`カラムにブルームフィルターインデックスを追加します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1,k2,v1");
    ```

- 次のステートメントは、`k2`カラムのブルームフィルターインデックスを削減します。
  
    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "k1");
    ```

- 次のステートメントは、`table1`の全てのブルームフィルターインデックスを削除します。

    ```SQL
    ALTER TABLE table1 SET ("bloom_filter_columns" = "");
    ```

> 注: インデックスの変更は非同期操作です。この操作の進行状況は、[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)を実行することで確認できます。一度に一つのテーブルに対して一つのインデックス変更タスクのみ実行可能です。
