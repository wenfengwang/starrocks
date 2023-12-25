---
displayed_sidebar: Chinese
---

# CREATE TABLE

## 機能

このステートメントは、テーブルを作成するために使用されます。

> **注意**
>
> この操作には、対応するデータベース内でのテーブル作成権限（CREATE TABLE）が必要です。

## 構文

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...]
[, index_definition1[, index_definition2, ...]])
[ENGINE = [olap|mysql|elasticsearch|hive|iceberg|hudi|jdbc]]
[key_desc]
[COMMENT "table comment"]
[partition_desc]
[distribution_desc]
[rollup_index]
[ORDER BY (column_definition1,...)]
[PROPERTIES ("key"="value", ...)]
[BROKER PROPERTIES ("key"="value", ...)]
```

## パラメータ説明

データベース名、テーブル名、列名などの変数を指定する際に、予約キーワードを使用した場合はバッククォート (`) で囲む必要があります。そうしないとエラーが発生する可能性があります。StarRocks の予約キーワードのリストについては、[キーワード](../keywords.md#保留关键字)を参照してください。

### **column_definition**

構文：

```sql
col_name col_type [agg_type] [NULL | NOT NULL] [DEFAULT "default_value"] [AUTO_INCREMENT] [AS generation_expr]
```

説明：

**col_name**：列名

注意：通常、`__op` や `__row` で始まる列名を直接作成することはできません。これらの列名は StarRocks によって特別な目的で予約されており、そのような列を作成すると予期せぬ動作が発生する可能性があります。このような列を作成する場合は、FE の動的パラメータ [`allow_system_reserved_names`](../../../administration/FE_configuration.md#allow_system_reserved_names) を `TRUE` に設定する必要があります。

**col_type**：列のデータ型

サポートされる列の型とその範囲は以下の通りです：

* TINYINT（1バイト）
  範囲：-2^7 + 1 ～ 2^7 - 1

* SMALLINT（2バイト）
  範囲：-2^15 + 1 ～ 2^15 - 1

* INT（4バイト）
  範囲：-2^31 + 1 ～ 2^31 - 1

* BIGINT（8バイト）
  範囲：-2^63 + 1 ～ 2^63 - 1

* LARGEINT（16バイト）
  範囲：-2^127 + 1 ～ 2^127 - 1

* FLOAT（4バイト）
  科学記数法がサポートされています。

* DOUBLE（8バイト）
  科学記数法がサポートされています。

* DECIMAL[(precision, scale)]（16バイト）
  精度を保証する小数点型。デフォルトは DECIMAL(10, 0) です。
    精度：1 ～ 38
    スケール：0 ～ 精度
  整数部分は：precision - scale
  科学記数法はサポートされていません。

* DATE（3バイト）
  範囲：0000-01-01 ～ 9999-12-31

* DATETIME（8バイト）
  範囲：0000-01-01 00:00:00 ～ 9999-12-31 23:59:59

* CHAR[(length)]

  固定長文字列。長さの範囲：1 ～ 255。デフォルトは 1 です。

* VARCHAR[(length)]

  可変長文字列。単位：バイト、デフォルト値は `1` です。
  * StarRocks 2.1.0 以前のバージョンでは、`length` の範囲は 1～65533 です。
  * 【公開テスト中】StarRocks 2.1.0 以降のバージョンでは、`length` の範囲は 1～1048576 です。

* HLL（1～16385バイト）

  HLL 列型は、長さとデフォルト値を指定する必要はありません。長さはデータの集約度によってシステム内で制御され、HLL 列は専用の [hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md)、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) を使用してクエリするか使用することができます。

* BITMAP
  BITMAP 列型は、長さとデフォルト値を指定する必要はありません。整数の集合を表し、要素の最大数は 2^64 - 1 までサポートされています。

* ARRAY
  配列内に子配列をネストすることができ、最大で 14 レベルまでネストできます。ARRAY の要素型を宣言するには、山括弧（< と >）を使用する必要があります。例：ARRAY < INT >。現在、配列の要素を [Fast Decimal](../data-types/DECIMAL.md) 型として宣言することはサポートされていません。

**agg_type**：集約型。指定されていない場合、その列は key 列です。そうでなければ、その列は value 列です。

```plain
サポートされる集約型は以下の通りです：

* SUM、MAX、MIN、REPLACE

* HLL_UNION（HLL 列専用の集約方法です）。

* BITMAP_UNION（BITMAP 列専用の集約方法です）。

* REPLACE_IF_NOT_NULL：この集約型の意味は、新しくインポートされたデータが NULL でない場合にのみ置換が行われるということです。新しくインポートされたデータが NULL の場合、StarRocks は元の値を保持します。
```

注意：

1. BITMAP_UNION 集約型の列は、インポート時の元のデータ型が `TINYINT, SMALLINT, INT, BIGINT` である必要があります。
2. テーブル作成時に `REPLACE_IF_NOT_NULL` 列が NOT NULL と指定されている場合、StarRocks はそれを NULL に変換しますが、ユーザーにはエラーを報告しません。ユーザーはこの型を利用して「部分列インポート」機能を実現できます。
  この型は集約モデル (`key_desc` の `type` が `AGGREGATE KEY`) にのみ有用です。

**NULL | NOT NULL**：列データが `NULL` を許容するかどうか。明細モデル、集約モデル、更新モデルのテーブルのすべての列はデフォルトで `NULL` と指定されています。主キーモデルのテーブルでは、指標列はデフォルトで `NULL`、次元列はデフォルトで `NOT NULL` と指定されています。ソースデータファイルに `NULL` 値が存在する場合は、`\N` で表すことができ、インポート時に StarRocks はそれを `NULL` として解析します。

**DEFAULT "default_value"**：列データのデフォルト値。データをインポートする際、その列に対応するソースデータファイルのフィールドが空の場合、`DEFAULT` キーワードで指定されたデフォルト値で自動的に埋められます。以下の3つの指定方法がサポートされています：

* **DEFAULT current_timestamp**：デフォルト値は現在の時刻です。[current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md) を参照してください。
* **DEFAULT `<デフォルト値>`**：指定された型の値がデフォルト値です。例えば、列の型が VARCHAR の場合、デフォルト値として `DEFAULT "beijing"` を指定できます。現在、ARRAY、BITMAP、JSON、HLL、BOOLEAN 型をデフォルト値として指定することはサポートされていません。
* **DEFAULT (`<式>`)**：デフォルト値は指定された関数の結果です。現在、[uuid()](../../sql-functions/utility-functions/uuid.md) と [uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md) の式のみがサポートされています。

**AUTO_INCREMENT**：自動増分列を指定します。自動増分列のデータ型は BIGINT のみをサポートし、自動増分 ID は 1 から始まり、増分ステップは 1 です。自動増分列の詳細については、[AUTO_INCREMENT](../auto_increment.md) を参照してください。v3.0 から、StarRocks はこの機能をサポートしています。

**AS generation_expr**：生成列とその式を指定します。[生成列](../generated_columns.md) は、式の結果を事前に計算して保存するために使用され、複雑な式を含むクエリの速度を向上させることができます。v3.1 から、StarRocks はこの機能をサポートしています。

### **index_definition**

テーブル作成時には、Bitmap インデックスのみを作成することがサポートされています。パラメータの説明と使用制限については、[Bitmap インデックス](../../../using_starrocks/Bitmap_index.md#创建索引) を参照してください。

```sql
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] [COMMENT '']
```

### **ENGINE の種類**

デフォルトは `olap` で、StarRocks の内部テーブルが作成されます。

選択肢：`mysql`、`elasticsearch`、`hive`、`jdbc`（2.3 以降）、`iceberg`、`hudi`（2.2 以降）。選択肢が指定された場合、対応するタイプの外部テーブル（external table）が作成され、テーブル作成時に CREATE EXTERNAL TABLE を使用する必要があります。詳細については、[外部テーブル](../../../data_source/External_table.md) を参照してください。

**3.0 バージョンから、Hive、Iceberg、Hudi、JDBC データソースのクエリには、Catalog を直接使用することが推奨され、外部テーブルの方法は推奨されなくなりました。詳細は [Hive catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)、[JDBC catalog](../../../data_source/catalog/jdbc_catalog.md) を参照してください。**

**3.1 バージョンから、Iceberg catalog 内で直接テーブルを作成することがサポートされています（現在は Parquet 形式のテーブルのみ）。[INSERT INTO](../data-manipulation/INSERT.md) を使用して Iceberg テーブルにデータを挿入できます。詳細は [Iceberg テーブルの作成](../../../data_source/catalog/iceberg_catalog.md#创建-iceberg-表) を参照してください。**

**3.2 バージョンから、Hive catalog 内で直接テーブルを作成することがサポートされています（現在は Parquet 形式のテーブルのみ）。[INSERT INTO](../data-manipulation/INSERT.md) を使用して Hive テーブルにデータを挿入できます。詳細は [Hive テーブルの作成](../../../data_source/catalog/hive_catalog.md#创建-hive-表) を参照してください。**

1. mysql の場合、properties で以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
        "user" = "your_user_name",
        "password" = "your_password",
        "database" = "database_name",
        "table" = "table_name"
    )
    ```

    注意：
    "table" の項目にある "table_name" は MySQL の実際のテーブル名です。
    CREATE TABLE ステートメントの table_name は、StarRocks 内の MySQL テーブルの名前であり、異なることがあります。

    StarRocks で MySQL テーブルを作成する目的は、StarRocks を通じて MySQL データベースにアクセスすることです。
    StarRocks 自体は、MySQL のデータを保守または保存しません。

2. elasticsearch の場合、properties で以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
        "user" = "root",
        "password" = "root",
        "index" = "tindex",
        "type" = "doc"
    )
```
    ```

    其中 `hosts` は Elasticsearch クラスタの接続アドレスで、一つまたは複数を指定できます。`user` と `password` は基本認証を有効にした Elasticsearch クラスタのユーザー名とパスワードです。`index` は StarRocks のテーブルに対応する Elasticsearch のインデックス名で、エイリアスも使用できます。`type` はインデックスのタイプを指定し、デフォルトは `doc` です。

3. Hive の場合、properties に以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "database" = "hive_db_name",
        "table" = "hive_table_name",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
    )
    ```

    ここで `database` は Hive テーブルに対応するデータベース名、`table` は Hive テーブルの名前、`hive.metastore.uris` は Hive metastore サービスのアドレスです。

4. JDBC の場合、properties に以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "resource"="jdbc0",
        "table"="dest_tbl"
    )
    ```

    ここで `resource` は使用する JDBC リソースの名前です。`table` は目的のデータベーステーブル名です。

5. Iceberg の場合、properties に以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "resource" = "iceberg0", 
        "database" = "iceberg", 
        "table" = "iceberg_table"
    )
    ```

    ここで `resource` は参照する Iceberg リソースの名前です。`database` は Iceberg テーブルが属するデータベースの名前です。`table` は Iceberg テーブルの名前です。
6. Hudi の場合、properties に以下の情報を提供する必要があります：

    ```sql
    PROPERTIES (
        "resource" = "hudi0", 
        "database" = "hudi", 
        "table" = "hudi_table" 
    )
    ```

    ここで `resource` は Hudi リソースの名前です。`database` は Hudi テーブルが属するデータベースの名前です。`table` は Hudi テーブルの名前です。

### **key_desc**

構文：

```sql
`key_type(k1[,k2 ...])`
```

> 説明
>
データは指定された key 列でソートされ、`key_type` によって異なる特性を持ちます。
`key_type` は以下のタイプをサポートしています：

* AGGREGATE KEY: key 列が同じレコードは、指定された集約タイプで value 列を集約します。レポートや多次元分析などのビジネスシナリオに適しています。
* UNIQUE KEY/PRIMARY KEY: key 列が同じレコードは、インポート順に value 列が上書きされます。key 列での増減改查を行うポイントクエリ（point query）ビジネスに適しています。
* DUPLICATE KEY: key 列が同じレコードは、StarRocks に同時に存在し、明細データの保存や集約特性のないビジネスシナリオに適しています。

デフォルトは DUPLICATE KEY で、データは key 列によってソートされます。

AGGREGATE KEY 以外の `key_type` では、テーブル作成時に `value` 列に集約タイプ（agg_type）を指定する必要はありません。

### コメント

テーブルのコメントはオプションです。テーブル作成時に COMMENT は `key_desc` の後になければならず、そうでないとテーブル作成に失敗します。

後でテーブルのコメントを変更したい場合は、`ALTER TABLE <table_name> COMMENT = "new table comment"`（バージョン3.1からサポート）を使用できます。

### **partition_desc**

[式分割](../../../table_design/expression_partitioning.md)（推奨）、[Range 分割](../../../table_design/Data_distribution.md#range-分区)、および [List 分割](../../../table_design/list_partitioning.md)の3種類の分割方式をサポートしています。

Range 分割を使用する場合、以下の3つの作成方法が提供され、その構文、説明、および例は以下の通りです：

* **動的分割**

    [動的分割](../../../table_design/dynamic_partitioning.md)は分割のライフサイクル管理（TTL）を提供します。StarRocks は自動的に新しい分割を事前に作成し、期限切れの分割を削除して、データの時宜性を保証します。この機能を有効にするには、テーブル作成時に動的分割に関連する属性を設定できます。

* **手動で分割を作成**

  * 各分割の上限のみを指定

    構文：

    ```sql
    PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
    PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
    [ ,
    PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
    , ... ] 
    )
    ```

    説明：

    指定された key 列と数値範囲を使用して分割します。

    * 分割名はアルファベットで始まり、アルファベット、数字、アンダースコアで構成される必要があります。
    * Range 分割列として以下のタイプの列のみがサポートされています：`TINYINT, SMALLINT, INT, BIGINT, LARGEINT, DATE, DATETIME`。
    * 分割は左閉右開の範囲で、最初の分割の左境界は最小値です。
    * NULL 値は **最小値** を含む分割にのみ格納されます。最小値を含む分割が削除された後、NULL 値はインポートできなくなります。
    * 一つまたは複数の列を分割列として指定できます。分割値が省略された場合、デフォルトで最小値が補充されます。
    * 一つの列のみを分割列として指定する場合、最後の分割の分割列の上限を MAXVALUE に設定できます。

    注意：

    1. 分割は一般に時間次元のデータ管理に使用されます。
    2. データのバックトラッキングが必要な場合は、最初の分割を空の分割として検討し、後で分割を追加できるようにすることができます。

    例：

    1. 分割列 `pay_dt` が DATE タイプで、日単位で分割されています。

        ```sql
        PARTITION BY RANGE(pay_dt)
        (
            PARTITION p1 VALUES LESS THAN ("2021-01-02"),
            PARTITION p2 VALUES LESS THAN ("2021-01-03"),
            PARTITION p3 VALUES LESS THAN ("2021-01-04")
        )
        ```

    2. 分割列 `pay_dt` が INT タイプで、日単位で分割されています。

        ```sql
        PARTITION BY RANGE(pay_dt)
        (
            PARTITION p1 VALUES LESS THAN ("20210102"),
            PARTITION p2 VALUES LESS THAN ("20210103"),
            PARTITION p3 VALUES LESS THAN ("20210104")
        )
        ```

    3. 分割列 `pay_dt` が INT タイプで、日単位で分割されており、最後の分割に上限がありません。

        ```sql
        PARTITION BY RANGE(pay_dt)
        (
            PARTITION p1 VALUES LESS THAN ("20210102"),
            PARTITION p2 VALUES LESS THAN ("20210103"),
            PARTITION p3 VALUES LESS THAN MAXVALUE
        )
        ```

  * 各分割の上限と下限を指定

    構文：

    ```sql
    PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
    (
        PARTITION <partition_name1> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
        [,
        PARTITION <partition_name2> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
        , ...]
    )
    ```

    説明：

    * 上限のみを指定する場合と比較して、各分割の上限と下限を指定する方が柔軟性があり、左右の区間を自由に定義できます。
    * その他の点では LESS THAN と同様です。
    * 一つの列のみを分割列として指定する場合、最後の分割の分割列の上限を MAXVALUE に設定できます。

    例：

    1. 分割列 `pay_dt` が DATE タイプで、月単位で分割されています。

        ```sql
        PARTITION BY RANGE (pay_dt)
        (
            PARTITION p202101 VALUES [("2021-01-01"), ("2021-02-01")),
            PARTITION p202102 VALUES [("2021-02-01"), ("2021-03-01")),
            PARTITION p202103 VALUES [("2021-03-01"), ("2021-04-01"))
        )
        ```

    2. 分割列 `pay_dt` が INT タイプで、月単位で分割されています。

        ```sql
        PARTITION BY RANGE (pay_dt)
        (
            PARTITION p202101 VALUES [("20210101"), ("20210201")),
            PARTITION p202102 VALUES [("20210201"), ("20210301")),
            PARTITION p202103 VALUES [("20210301"), ("20210401"))
        )
        ```

    3. 分割列 `pay_dt` が INT タイプで、月単位で分割されており、最後の分割に上限がありません。

        ```sql
        PARTITION BY RANGE (pay_dt)
        (
            PARTITION p202101 VALUES [("20210101"), ("20210201")),
            PARTITION p202102 VALUES [("20210201"), ("20210301")),
            PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
        )

* **一括で分割を作成**

  構文：

  * 分割列が時間タイプの場合

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

    * 分割列が整数タイプの場合

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

    説明：
    START 値、END 値、および EVERY 句を使用して分割の増分値を定義することにより、ユーザーは一括で分割を生成できます。

    * 現在、分割列は日付タイプと整数タイプのみをサポートしています。
    * 分割列が日付タイプの場合、`INTERVAL` キーワードを指定して日付間隔を表します。現在、日付間隔は hour (v3.0)、day、week、month、year をサポートしており、分割の命名規則は動的分割と同じです。
    * 分割列が整数タイプの場合、START 値と END 値は引用符で囲む必要があります。
    * 一つの列のみを分割列として指定できます。

    詳細は[一括で分割を作成](../../../table_design/Data_distribution.md#range-分区)を参照してください。

    例：

    1. 分割列 `pay_dt` が DATE タイプで、年単位で分割されています。

        ```sql
        PARTITION BY RANGE (pay_dt) (
            START ("2018-01-01") END ("2023-01-01") EVERY (INTERVAL 1 YEAR)
        )
        ```

    2. 分割列 `pay_dt` が INT タイプで、年単位で分割されています。

        ```sql
        PARTITION BY RANGE (pay_dt) (
            START ("2018") END ("2023") EVERY (1)
        )
        ```

### **distribution_desc**


サポートされている分割方法は、ランダム分割（Random bucketing）とハッシュ分割（Hash bucketing）です。分割情報が指定されていない場合、StarRocksはデフォルトでランダム分割を使用し、分割数を自動的に設定します。

* ランダム分割（v3.1以降）

  各パーティションのデータに対して、StarRocksはデータを特定の列の値に影響を受けずにランダムにすべてのバケットに分散させます。また、システムによる分割数の設定を選択した場合は、分割情報を設定する必要はありません。分割数を手動で指定する場合は、次の構文を使用します：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <num>
  ```

  ただし、大量のデータをクエリし、クエリで特定の列を頻繁に条件として使用する場合、ランダム分割によるクエリのパフォーマンスは十分に理想的ではない場合があります。このシナリオでは、ハッシュ分割を使用することをお勧めします。これにより、クエリで頻繁に使用されるこれらの列を条件として持つ少数のバケットのみをスキャンおよび計算する必要があり、クエリのパフォーマンスが大幅に向上します。

  **注意事項**

  * プライマリキーモデルテーブル、更新モデルテーブル、および集計テーブルはサポートされていません。
  * [Colocation Group](../../../using_starrocks/Colocate_join.md)の指定はサポートされていません。
  * [Spark Load](../../../loading/SparkLoad.md)はサポートされていません。
  * 2.5.7以降、テーブルを作成する際に**分割数を手動で指定する必要はありません**。StarRocksは分割数を自動的に設定します。分割数を手動で設定する場合は、[分割数の設定](../../../table_design/Data_distribution.md#分割数の設定)を参照してください。

  詳細なランダム分割の情報については、[ランダム分割](../../../table_design/Data_distribution.md#ランダム分割v31以降)を参照してください。

* ハッシュ分割

  構文：

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  各パーティションのデータは、ハッシュ分割キーと分割数に基づいてStarRocksによってハッシュ分割されます。

  分割キーの選択については、列が高基数であり、クエリ条件として頻繁に使用される場合は、分割キーとして優先的に選択する必要があります。両方の条件を満たす列が存在しない場合は、クエリに基づいて判断する必要があります。
  * クエリが複雑な場合は、データの均等な分散とクラスタリソースの利用率の向上を保証するため、高基数の列を分割キーとして選択することをお勧めします。
  * クエリが単純な場合は、クエリの効率を向上させるために、頻繁にクエリ条件として使用される列を分割キーとして選択することをお勧めします。
  また、データの偏りが深刻な場合は、データの分割キーとして複数の列を使用することもできますが、3つ以上の列を使用することはお勧めしません。
  分割キーの選択についての詳細は、[分割キーの選択](../../../table_design/Data_distribution.md#ハッシュ分割)を参照してください。

  **注意事項**

  * **テーブルを作成する際に、分割キーを指定する必要があります**。
  * 分割キーとして指定された列の値は更新できません。
  * 分割キーが指定された後は変更できません。
  * 2.5.7以降、テーブルを作成する際に**分割数を手動で指定する必要はありません**。StarRocksは分割数を自動的に設定します。分割数を手動で設定する場合は、[分割数の設定](../../../table_design/Data_distribution.md#分割数の設定)を参照してください。

### **ORDER BY**

3.0以降、プライマリキーモデルはプライマリキーとソートキーを分離し、ソートキーは `ORDER BY` で指定され、任意の列の組み合わせで構成されます。
> **注意**
>
> ソートキーが指定されている場合、ソートキーに基づいてプレフィックスインデックスが構築されます。ソートキーが指定されていない場合、プライマリキーに基づいてプレフィックスインデックスが構築されます。

### **プロパティ**

#### データの初期ストレージメディア、自動冷却時間、およびレプリカ数の設定

ENGINEタイプがOLAPの場合、属性 `properties` でテーブルのデータの初期ストレージメディア（storage_medium）、自動冷却時間（storage_cooldown_time）または間隔（storage_cooldown_ttl）、およびレプリカ数（replication_num）を設定できます。

属性の有効範囲：テーブルが単一のパーティションテーブルの場合、上記の属性はテーブルの属性です。テーブルが複数のパーティションに分割されている場合、上記の属性は各パーティションに属します。さらに、異なるパーティションに異なる属性を持たせたい場合は、テーブル作成後に [ALTER TABLE ... ADD PARTITION または ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md) を実行できます。

**データの初期ストレージメディア、自動冷却時間の設定**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

* `storage_medium`：データの初期ストレージメディアで、`SSD` または `HDD` のいずれかを指定します。このパラメータを明示的に指定する場合は、クラスタのストレージメディアを指定した BE の静的パラメータ `storage_root_path` と一致することを確認してください。
  
  FEの設定項目 `enable_strict_storage_medium_check` が `true` の場合、テーブル作成時に BE のストレージメディアが厳密にチェックされます。テーブル作成文のストレージメディアと BE のストレージタイプが一致しない場合、テーブル作成文は `Failed to find enough hosts with storage medium [SSD|HDD] at all backends...` というエラーが発生します。`enable_strict_storage_medium_check` が `false` の場合、このエラーを無視してテーブルを作成することができますが、後でクラスタのディスクスペースのバランスが崩れる可能性があります。

  2.3.6、2.4.2、2.5.1、3.0 以降では、明示的にこのパラメータを指定しない場合、システムがストレージメディアを自動的に推論します。

  * 以下のシナリオでは、システムは SSD と推論します：
    * BE が報告するストレージパス (`storage_root_path`) がすべて SSD の場合。
    * BE が報告するストレージパスに SSD と HDD の両方が含まれている場合。さらに、2.3.10、2.4.5、2.5.4、3.0 以降では、BE が報告するストレージパスの両方があり、FE の設定ファイルで `storage_cooldown_second` が設定されている場合も該当します。
  * 以下のシナリオでは、システムは HDD と推論します：
    * BE が報告するストレージパスがすべて HDD の場合。
    * 2.3.10、2.4.5、2.5.4、3.0 以降では、BE が報告するストレージパスの両方があり、FE の設定ファイルで `storage_cooldown_second` が設定されていない場合も該当します。

* `storage_cooldown_ttl` または `storage_cooldown_time`：データの自動冷却間隔または時間ポイント。データの自動冷却は、データをSSDからHDDに自動的に移動することを意味します。データの初期ストレージメディアがSSDの場合にのみ有効です。

  **パラメータの説明**

  * `storage_cooldown_ttl`：このテーブルの分割の自動冷却**間隔**。最新のいくつかの分割をSSDに保持し、それ以外の古い分割を一定の時間間隔でHDDに自動的に冷却する場合は、このパラメータを使用できます。各分割の自動冷却時間ポイントは、このパラメータの値 + 分割の上限時間です。
  
     `<num> YEAR`、`<num> MONTH`、`<num> DAY`、または `<num> HOUR` のいずれかの値を取ります。`<num>` は非負の整数です。デフォルト値は空で、このテーブルの分割は自動冷却されません。
  
     たとえば、テーブル作成時に `"storage_cooldown_ttl"="1 DAY"` を指定し、分割 `p20230801` が存在する場合、範囲は `[2023-08-01 00:00:00,2023-08-02 00:00:00)` であり、この分割の自動冷却時間ポイントは `2023-08-03 00:00:00`、つまり `2023-08-02 00:00:00 + 1 DAY` です。テーブル作成時に `"storage_cooldown_ttl"="0 DAY"` を指定した場合、この分割の自動冷却時間ポイントは `2023-08-02 00:00:00` です。

  * `storage_cooldown_time`：このテーブルの自動冷却**時間ポイント**（絶対時間）。データはこの時間ポイント以降、SSDからHDDに自動的に冷却されます。設定する時間は現在の時間よりも大きくする必要があります。値の形式は "yyyy-MM-dd HH:mm:ss" です。異なる分割に異なる自動冷却時間ポイントを持たせる場合は、[ALTER TABLE ... ADD PARTITION または ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md) を実行する必要があります。

  **使用方法**

  * 現在、StarRocksは次のデータの自動冷却に関連するパラメータを提供しています。比較は次のとおりです：
    * `storage_cooldown_ttl`：テーブルの属性で、テーブル内の分割の自動冷却時間間隔を指定し、システムが到達時間ポイント（時間間隔+分割の時間上限）の分割を自動的に冷却します。テーブルは分割の粒度で自動的に冷却されるため、より柔軟です。
    * `storage_cooldown_time`：テーブルの属性で、テーブルの自動冷却時間ポイント（絶対時間）を指定します。テーブル作成後、異なる分割に異なる時間ポイントを設定することもできます。
    * `storage_cooldown_second`：FEの静的パラメータで、クラスタ全体のすべてのテーブルの自動冷却遅延を指定します。
  * テーブル属性 `storage_cooldown_ttl` または `storage_cooldown_time` は、FEの静的パラメータ `storage_cooldown_second` よりも優先されます。
  * 上記のパラメータを設定する場合、`"storage_medium = "SSD"``を指定する必要があります。
  * 上記のパラメータを設定しない場合、自動冷却は行われません。
  * `SHOW PARTITIONS FROM <table_name>` を実行して各分割の自動冷却時間ポイントを確認します。

  **制限事項**
  * 式パーティションとリストパーティションはサポートされていません。
  * パーティション列は日付型以外にはサポートされていません。
  * 複数のパーティション列はサポートされていません。
  * プライマリキーモデルテーブルはサポートされていません。

**パーティションのTabletのレプリカ数の設定**

`replication_num`：パーティションのTabletのレプリカ数。デフォルトは3です。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### テーブル作成時に列にBloom Filterインデックスを追加する

OLAPエンジンの場合、特定の列にBloom Filterインデックスを追加できます。Bloom Filterインデックスの使用には以下の制限があります:


* 主键模型と明細モデルでは、すべての列にBloom filterインデックスを作成できます。集約モデルと更新モデルでは、ディメンション列（つまり、キー列）のみがBloom filterインデックスの作成をサポートしています。
* TINYINT、FLOAT、DOUBLE、DECIMAL型の列にはBloom filterインデックスを作成できません。
* Bloom filterインデックスは、`in`および`=`のクエリ条件の効率を向上させることができます。値が分散しているほど効果が高くなります。

詳細については、[Bloom filterインデックス](../../../using_starrocks/Bloomfilter_index.md)を参照してください。

```sql
PROPERTIES (
    "bloom_filter_columns" = "k1, k2, k3"
)
```

#### Colocate Joinをサポートするプロパティを追加

Colocate Join機能を使用する場合は、プロパティで次のように指定する必要があります:

``` sql
PROPERTIES (
    "colocate_with" = "table1"
)
```

Colocate Joinの詳細な使用方法とアプリケーションシナリオについては、[Colocate Join](../../../using_starrocks/Colocate_join.md)セクションを参照してください。

#### ダイナミックパーティションの設定

ダイナミックパーティション機能を使用する場合は、プロパティで次のパラメータを指定する必要があります:

``` sql
PROPERTIES (
    "dynamic_partition.enable" = "true|false",
    "dynamic_partition.time_unit" = "DAY|WEEK|MONTH",
    "dynamic_partition.start" = "${integer_value}",
    "dynamic_partition.end" = "${integer_value}",
    "dynamic_partition.prefix" = "${string_value}",
    "dynamic_partition.buckets" = "${integer_value}"
)
```

| パラメータ                     | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| `dynamic_partition.enable`    | 任意 | ダイナミックパーティション機能を有効にする場合は、`TRUE`（デフォルト）または`FALSE`を指定します。 |
| `dynamic_partition.time_unit` | 必須 | ダイナミックパーティションの時間単位を指定します。`DAY`、`WEEK`、または`MONTH`のいずれかを指定します。時間単位によって、動的に作成されるパーティション名のサフィックスの形式が決まります。 <br />`DAY`を指定する場合、動的に作成されるパーティション名のサフィックスの形式はyyyyMMddです。例えば、20200321です。<br />`WEEK`を指定する場合、動的に作成されるパーティション名のサフィックスの形式はyyyy_wwです。例えば、2020_13は2020年の13週目を表します。<br />`MONTH`を指定する場合、動的に作成されるパーティション名のサフィックスの形式はyyyyMMです。例えば、202003です。 |
| `dynamic_partition.start`    | 任意 | 保持するダイナミックパーティションの開始オフセットを指定します。値の範囲は負の整数です。`dynamic_partition.time_unit`のプロパティによって、当日（週/月）を基準に、このオフセットより前のパーティション範囲が削除されます。例えば、-3を設定し、`dynamic_partition.time_unit`が`day`の場合、3日前のパーティションが削除されます。<br />指定しない場合は、デフォルトで`Integer.MIN_VALUE`、つまり-2147483648、つまり過去のパーティションは削除されません。 |
| `dynamic_partition.end`      | 必須 | 事前に作成するパーティションの数を指定します。値の範囲は正の整数です。`dynamic_partition.time_unit`のプロパティによって、当日（週/月）を基準に、対応する範囲のパーティションが事前に作成されます。 |
| `dynamic_partition.prefix`   | 任意 | ダイナミックパーティションの接頭辞名を指定します。デフォルト値は`p`です。 |
| `dynamic_partition.buckets`  | 任意 | ダイナミックパーティションのバケット数です。デフォルトは、BUCKETS予約語で指定されたバケット数、またはStarRocksが自動的に設定したバケット数と同じです。 |

#### ランダムバケットテーブルのバケットサイズの設定

バージョン3.2以降、ランダムバケットのテーブルでは、`PROPERTIES`で`bucket_size`パラメータを設定してバケットサイズを指定し、必要に応じて動的にバケット数を増やすことができます。単位はBです。

``` sql
PROPERTIES (
    "bucket_size" = "1073741824"
)
```

#### データ圧縮アルゴリズムの設定

テーブルを作成する際に、`compression`属性を追加してデータ圧縮アルゴリズムを指定することができます。

`compression`の有効な値は次のとおりです：

* `LZ4`：LZ4アルゴリズム。
* `ZSTD`：Zstandardアルゴリズム。
* `ZLIB`：zlibアルゴリズム。
* `SNAPPY`：Snappyアルゴリズム。

データ圧縮アルゴリズムを指定しない場合、StarRocksはデフォルトでLZ4を使用します。

適切なデータ圧縮アルゴリズムの選択方法については、[データ圧縮](../../../table_design/data_compression.md)を参照してください。

#### データインポートのセキュリティレベルの設定

StarRocksクラスタに複数のデータレプリカがある場合、データインポートのセキュリティレベルを指定するために、`PROPERTIES`で`write_quorum`パラメータを設定することができます。これにより、データのインポートが成功したデータレプリカの数を指定し、StarRocksがインポートが成功したと判断することができます。このプロパティは2.5バージョンからサポートされています。

`write_quorum`の値と説明は次のとおりです：

* `MAJORITY`：デフォルト値。**過半数**のデータレプリカがインポートに成功した場合、StarRocksはインポートが成功したと返し、それ以外の場合は失敗と返します。
* `ONE`：**1つの**データレプリカがインポートに成功した場合、StarRocksはインポートが成功したと返し、それ以外の場合は失敗と返します。
* `ALL`：**すべて**のデータレプリカがインポートに成功した場合、StarRocksはインポートが成功したと返し、それ以外の場合は失敗と返します。

> **注意**
>
> * 低いデータインポートのセキュリティレベルを設定すると、データのアクセス不可やデータの損失のリスクが増加します。例えば、StarRocksクラスタに2つのデータレプリカがあり、`write_quorum`を`ONE`に設定した場合、実際には1つのレプリカのみが正常にインポートされ、そのレプリカが後でオフラインになった場合、そのテーブルのデータにアクセスできなくなります。サーバーのディスクが損傷した場合、そのテーブルのデータが失われます。
> * すべてのデータレプリカがインポートのステータスを返した後、StarRocksはインポートタスクのステータスを返します。ステータスが不明なレプリカがある場合、StarRocksはインポートタスクのステータスを返しません。タイムアウトしたレプリカも失敗としてマークされます。

#### 複数のレプリカ間でのデータの書き込みと同期方法の指定

StarRocksクラスタに複数のデータレプリカがある場合、`PROPERTIES`で`replicated_storage`パラメータを設定して、データの書き込みと同期方法を指定することができます。

* `true`に設定する（3.0以降のデフォルト値）と、Single Leader Replicationが有効になります。つまり、データはプライマリレプリカにのみ書き込まれ、プライマリレプリカがデータをセカンダリレプリカに同期します。このモードは、複数のレプリカによる書き込みに伴うCPUコストを効果的に削減することができます。このモードは2.5バージョンからサポートされています。
* `false`に設定する（2.5バージョンのデフォルト値）と、leaderless replicationが有効になります。つまり、データは複数のレプリカに直接書き込まれ、プライマリとセカンダリの区別はありません。このモードはCPUコストが高くなります。

デフォルトの設定は、ほとんどのシナリオでより良い書き込み性能を実現できます。既存のテーブルの複数のレプリカの書き込みと同期方法を変更する場合は、ALTER TABLEコマンドを実行することができます。例：

```sql
ALTER TABLE example_db.my_table
SET ("replicated_storage" = "true");
```

#### テーブル作成時に複数のロールアップを一括作成

構文：

```sql
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key" = "value", ...)],...)
```

#### View Delta JoinクエリのためのUnique Keyと外部キー制約の定義

View Delta Joinシナリオでクエリのリライトを有効にするには、Delta JoinのテーブルにUnique Key制約`unique_constraints`と外部キー制約`foreign_key_constraints`を定義する必要があります。詳細については、[非同期マテリアライズドビュー - View Delta Joinシナリオでのクエリリライト](../../../using_starrocks/query_rewrite_with_materialized_views.md#view-delta-join-改写)を参照してください。

```SQL
PROPERTIES (
    "unique_constraints" = "<unique_key>[, ...]",
    "foreign_key_constraints" = "
    (<child_column>[, ...]) 
    REFERENCES 
    [catalog_name].[database_name].<parent_table_name>(<parent_column>[, ...])
    [;...]
    "
)
```

* `child_column`：現在のテーブルの外部キーカラム。複数の`child_column`を定義することができます。
* `catalog_name`：参照するテーブルが存在するデータカタログの名前。このパラメータを指定しない場合は、デフォルトのカタログが使用されます。
* `database_name`：参照するテーブルが存在するデータベースの名前。このパラメータを指定しない場合は、現在のデータベースが使用されます。
* `parent_table_name`：参照するテーブルの名前。
* `parent_column`：参照する列の名前。これは、対応するテーブルの主キーまたはユニークキーである必要があります。

> **注意**
>
> * `unique_constraints`制約と`foreign_key_constraints`制約は、クエリのリライトにのみ使用されます。データのインポート時に外部キー制約のチェックは行われません。インポートするデータが制約条件を満たしていることを確認する必要があります。
> * プライマリキーモデルテーブルのPrimary Keyまたは更新モデルテーブルのUnique Keyは、デフォルトで`unique_constraints`として設定されており、手動で設定する必要はありません。
> * `foreign_key_constraints`の`child_column`は、他のテーブルの`unique_constraints`の`unique_key`に対応する必要があります。
> * `child_column`と`parent_column`の数は一致している必要があります。
> * `child_column`と対応する`parent_column`のデータ型は一致している必要があります。

#### StarRocksストレージアナリティクスクラスタのためのクラウドネイティブテーブルの作成

[StarRocksストレージアナリティクスクラスタ](../../../deployment/shared_data/s3.md)を使用するためには、次のPROPERTIESを使用してクラウドネイティブテーブルを作成する必要があります。

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

* `datacache.enable`：ローカルディスクキャッシュを有効にするかどうかを指定します。デフォルト値：`true`。

  * このプロパティを`true`に設定すると、データはオブジェクトストレージ（またはHDFS）とローカルディスク（クエリの高速化のためのキャッシュとして）の両方にインポートされます。
  * このプロパティを`false`に設定すると、データはオブジェクトストレージにのみインポートされます。

  > **注意**
  >
  > ローカルディスクキャッシュを有効にする場合は、BEの`storage_root_path`設定でディスクディレクトリを指定する必要があります。詳細については、[BEの設定項目](../../../administration/BE_configuration.md)を参照してください。


```
* `datacache.partition_duration`：ホットデータの有効期間。ローカルディスクキャッシュが有効になっている場合、すべてのデータはローカルディスクキャッシュにインポートされます。キャッシュがいっぱいになると、StarRocksは最近あまり使われていない（Less recently used）データをキャッシュから削除します。削除されたデータをスキャンするクエリがある場合、StarRocksはそのデータが有効期間内にあるかどうかをチェックします。データが有効期間内であれば、StarRocksは再びデータをキャッシュにインポートします。データが有効期間外であれば、StarRocksはそれをキャッシュにインポートしません。この属性は文字列で、`YEAR`、`MONTH`、`DAY`、`HOUR`の単位で指定できます。例えば、`7 DAY`や`12 HOUR`です。指定がない場合、StarRocksはすべてのデータをホットデータとしてキャッシュします。

  > **説明**
  >
  > `datacache.enable`が`true`に設定されている場合のみ、この属性は使用できます。

* `enable_async_write_back`：データを非同期にオブジェクトストレージに書き込むことを許可するかどうか。デフォルト値：`false`。

  * この属性が`true`に設定されている場合、インポートタスクはデータがローカルディスクキャッシュに書き込まれた直後に成功として返され、データは非同期にオブジェクトストレージに書き込まれます。データの非同期書き込みを許可することでインポートのパフォーマンスを向上させることができますが、システムに障害が発生した場合、データの信頼性にリスクが生じる可能性があります。
  * この属性が`false`に設定されている場合、データがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後でのみ、インポートタスクは成功として返されます。データの非同期書き込みを無効にすることで、より高い可用性を保証しますが、インポートのパフォーマンスが低下する可能性があります。

#### Fast Schema Evolutionの設定

`fast_schema_evolution`：テーブルのfast schema evolutionを有効にするかどうか。値：`TRUE`または`FALSE`（デフォルト）。有効にすると、列の追加や削除時にschema changeの速度を向上させ、リソースの使用量を減らすことができます。現在、テーブル作成時にのみこの属性を有効にすることができ、テーブル作成後は[ALTER TABLE](../data-definition/ALTER_TABLE.md)を通じてこの属性を変更することはできません。バージョン3.2.0から、このパラメータがサポートされています。

> **注記**
>
> * StarRocksのストレージと計算が分離されたクラスターはこのパラメータをサポートしていません。
> * クラスター全体でこの設定を行いたい場合、例えばクラスター全体でfast schema evolutionを無効にする場合は、FEの動的パラメータ[`enable_fast_schema_evolution`](../../../administration/FE_configuration.md#enable_fast_schema_evolution)を設定できます。

## 例

### Hash分割表を作成し、key列に基づいてデータを集約する

Hash分割を使用し、列ストアで、同じkeyのレコードを集約するolapテーブルを作成します。

``` sql
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type" = "column");
```

### ストレージメディアとデータの自動冷却時間を設定してテーブルを作成する

Hash分割を使用し、列ストアで、同じkeyのレコードを上書きし、初期ストレージメディアと冷却時間を設定するolapテーブルを作成します。

``` sql
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT DEFAULT "10"
)
ENGINE = olap
UNIQUE KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type" = "column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2025-06-04 00:00:00"
);
```

または

``` sql
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT DEFAULT "10"
)
ENGINE = olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type" = "column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2025-06-04 00:00:00"
);
```

### パーティションテーブルを作成する

Rangeパーティションを使用し、Hash分割を使用し、デフォルトで列ストアを使用し、同じkeyのレコードが同時に存在し、初期ストレージメディアと冷却時間を設定するolapテーブルを作成します。

LESS THAN方式で範囲によるパーティションを分割します：

``` sql
CREATE TABLE example_db.table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE = olap
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1)
(
    PARTITION p1 VALUES LESS THAN ("2014-01-01"),
    PARTITION p2 VALUES LESS THAN ("2014-06-01"),
    PARTITION p3 VALUES LESS THAN ("2014-12-01")
)
DISTRIBUTED BY HASH(k2)
PROPERTIES(
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2025-06-04 00:00:00"
);
```

説明：
このステートメントは、以下の3つのパーティションにデータを分割します：

``` sql
( {    MIN     },   {"2014-01-01"} )
[ {"2014-01-01"},   {"2014-06-01"} )
[ {"2014-06-01"},   {"2014-12-01"} )
```

これらのパーティション範囲外のデータは無効なデータとしてフィルタリングされます。

Fixed Range方式でパーティションを作成します：

``` sql
CREATE TABLE table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE = olap
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1, k2, k3)
(
    PARTITION p1 VALUES [("2014-01-01", "10", "200"), ("2014-01-01", "20", "300")),
    PARTITION p2 VALUES [("2014-06-01", "100", "200"), ("2014-07-01", "100", "300"))
)
DISTRIBUTED BY HASH(k2)
PROPERTIES(
    "storage_medium" = "SSD"
);
```

### MySQL外部テーブルを作成する

``` sql
CREATE EXTERNAL TABLE example_db.table_mysql
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE = mysql
PROPERTIES
(
    "host" = "127.0.0.1",
    "port" = "8239",
    "user" = "mysql_user",
    "password" = "mysql_passwd",
    "database" = "mysql_db_test",
    "table" = "mysql_table_test"
);
```

### HLL列を含むテーブルを作成する

``` sql
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 HLL HLL_UNION,
    v2 HLL HLL_UNION
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type" = "column");
```

### `BITMAP_UNION`集約型を持つテーブルを作成する

v1とv2の列の元のデータ型は`TINYINT, SMALLINT, INT`でなければなりません。

``` sql
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 BITMAP BITMAP_UNION,
    v2 BITMAP BITMAP_UNION
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type" = "column");
```

### Colocate Joinをサポートする2つのテーブルを作成する

t1とt2の2つのテーブルを作成し、これらのテーブルはColocate Joinを行うことができます。2つのテーブルのプロパティの`colocate_with`の値は一致している必要があります。

``` sql
CREATE TABLE `t1` (
    `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) ENGINE = OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "colocate_with" = "t1"
);

CREATE TABLE `t2` (
    `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) ENGINE = OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "colocate_with" = "t1"
);
```

### bitmapインデックスを持つテーブルを作成する

``` sql
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM,
    INDEX k1_idx (k1) USING BITMAP COMMENT 'xxxxxx'
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type" = "column");
```

### 動的パーティションテーブルを作成する

動的パーティションテーブルを作成するには、FEの設定で**動的パーティション**機能（`"dynamic_partition.enable" = "true"`）を有効にする必要があります。パラメータは、本文の[動的パーティションの設定](#動的パーティションの設定)で参照できます。

このテーブルは毎日3日前のパーティションを事前に作成し、3日前のパーティションを削除します。例えば、今日が`2020-01-08`であれば、`p20200108`、`p20200109`、`p20200110`、`p20200111`という名前のパーティションが作成されます。パーティションの範囲は以下の通りです：

```plain text
[types: [DATE]; keys: [2020-01-08]; ‥types: [DATE]; keys: [2020-01-09]; )
[types: [DATE]; keys: [2020-01-09]; ‥types: [DATE]; keys: [2020-01-10]; )
[types: [DATE]; keys: [2020-01-10]; ‥types: [DATE]; keys: [2020-01-11]; )
[types: [DATE]; keys: [2020-01-11]; ‥types: [DATE]; keys: [2020-01-12]; )
```

```sql
CREATE TABLE example_db.dynamic_partition
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE = olap
DUPLICATE KEY(k1, k2, k3)
PARTITION BY RANGE (k1)
(
    PARTITION p1 VALUES LESS THAN ("2014-01-01"),
    PARTITION p2 VALUES LESS THAN ("2014-06-01"),
    PARTITION p3 VALUES LESS THAN ("2014-12-01")
)
DISTRIBUTED BY HASH(k2)
PROPERTIES(
    "storage_medium" = "SSD",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p"
);
```

### Hive外部テーブルを作成する

``` SQL

CREATE EXTERNAL TABLE example_db.table_hive
(
    k1 TINYINT,
    k2 VARCHAR(50),
    v INT
)
ENGINE = hive
PROPERTIES
(
    "resource" = "hive0",
    "database" = "hive_db_name",
    "table" = "hive_table_name"
);
```

### 主キーモデルのテーブルを作成し、ソートキーを指定する

地域や最近のアクティブ時間に基づいてユーザー状況をリアルタイムで分析する必要がある場合、ユーザーIDを表す `user_id` 列を主キーとし、地域を表す `address` 列と最近のアクティブ時間を表す `last_active` 列をソートキーとして指定できます。テーブル作成のSQLは以下の通りです：

```SQL
create table users (
    user_id bigint NOT NULL,
    name string NOT NULL,
    email string NULL,
    address string NULL,
    age tinyint NULL,
    sex tinyint NULL,
    last_active datetime,
    property0 tinyint NOT NULL,
    property1 tinyint NOT NULL,
    property2 tinyint NOT NULL,
    property3 tinyint NOT NULL
) 
PRIMARY KEY (`user_id`)
DISTRIBUTED BY HASH(`user_id`)
ORDER BY(`address`,`last_active`)
PROPERTIES(
    "replication_num" = "3",
    "enable_persistent_index" = "true"
);
```

## 参照文献

* [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
* [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
* [USE](USE.md)
* [ALTER TABLE](ALTER_TABLE.md)
* [DROP TABLE](DROP_TABLE.md)
