---
displayed_sidebar: "Japanese"
---

# CREATE TABLE（テーブルの作成）

## 説明

StarRocks に新しいテーブルを作成します。

> **注意**
>
> この操作には、宛先データベースで CREATE TABLE 権限が必要です。

## 構文

```plaintext
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...]
[, index_definition1[, index_definition12,]])
[ENGINE = [olap|mysql|elasticsearch|hive|hudi|iceberg|jdbc]]
[key_desc]
[COMMENT "table comment"]
[partition_desc]
[distribution_desc]
[rollup_index]
[ORDER BY (column_definition1,...)]
[PROPERTIES ("key"="value", ...)]
[BROKER PROPERTIES ("key"="value", ...)]
```

## パラメータ

### column_definition

構文:

```SQL
col_name col_type [agg_type] [NULL | NOT NULL] [DEFAULT "default_value"] [AUTO_INCREMENT] [AS generation_expr]
```

**col_name**: カラム名。

通常、`__op` または `__row` で始まるカラムは作成できません。これらの名前形式は StarRocks で特別な目的で予約されており、このようなカラムを作成すると未定義の動作が発生する可能性があります。このようなカラムを作成する必要がある場合は、FE ダイナミックパラメータ [`allow_system_reserved_names`](../../../administration/Configuration.md#allow_system_reserved_names) を `TRUE` に設定します。

**col_type**: カラムの型。次のような特定のカラム情報（型と範囲など）があります。

- TINYINT（1 バイト）: -2^7 + 1 から 2^7 - 1 までの範囲。
- SMALLINT（2 バイト）: -2^15 + 1 から 2^15 - 1 までの範囲。
- INT（4 バイト）: -2^31 + 1 から 2^31 - 1 までの範囲。
- BIGINT（8 バイト）: -2^63 + 1 から 2^63 - 1 までの範囲。
- LARGEINT（16 バイト）: -2^127 + 1 から 2^127 - 1 までの範囲。
- FLOAT（4 バイト）: 科学的表記法をサポートしています。
- DOUBLE（8 バイト）: 科学的表記法をサポートしています。
- DECIMAL[(precision, scale)]（16 バイト）

  - デフォルト値: DECIMAL(10, 0)
  - precision: 1 〜 38
  - scale: 0 〜 precision
  - 整数部: precision - scale

    科学的表記法はサポートされていません。

- DATE（3 バイト）: 0000-01-01 から 9999-12-31 までの範囲。
- DATETIME（8 バイト）: 0000-01-01 00:00:00 から 9999-12-31 23:59:59 までの範囲。
- CHAR[(length)]: 固定長の文字列。範囲: 1 〜 255。デフォルト値: 1。
- VARCHAR[(length)]: 可変長の文字列。デフォルト値は 1。単位: バイト。StarRocks 2.1 より前のバージョンでは、`length` の値の範囲は 1 〜 65533 です。[プレビュー] StarRocks 2.1 以降のバージョンでは、`length` の値の範囲は 1 〜 1048576 です。
- HLL（1〜16385 バイト）: HLL 型では、長さやデフォルト値を指定する必要はありません。データ集約に応じて、システム内で長さが制御されます。HLL カラムは、[hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md)、および [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) によってのみクエリや使用が可能です。
- BITMAP: ビットマップ型は、指定された長さやデフォルト値を必要としません。符号なしの bigint 数のセットを表します。最大の要素は 2^64 - 1 までです。

**agg_type**: 集計タイプ。指定されていない場合、このカラムはキーカラムです。
指定されている場合、値カラムです。サポートされている集計タイプは次のとおりです。

- SUM、MAX、MIN、REPLACE
- HLL_UNION（HLL タイプのみ）
- BITMAP_UNION（BITMAP タイプのみ）
- REPLACE_IF_NOT_NULL: これは、インポートされるデータが非 NULL 値の場合にのみ置換されることを意味します。NULL 値の場合は、StarRocks は元の値を保持します。

> 注意
>
> - 集計タイプが BITMAP_UNION のカラムをインポートする場合、元のデータ型は TINYINT、SMALLINT、INT、BIGINT である必要があります。
> - テーブル作成時に REPLACE_IF_NOT_NULL カラムで NOT NULL が指定されている場合、StarRocks はエラーレポートをユーザーに送信せずにデータを NULL に変換します。これにより、ユーザーは選択したカラムをインポートできます。

この集計タイプは、AGGREGATE KEY タイプの key_desc が AGGREGATE KEY の集計テーブルにのみ適用されます。

**NULL | NOT NULL**: カラムが `NULL` になることを許可するかどうか。デフォルトでは、Duplicate Key、Aggregate、または Unique Key テーブルを使用するテーブルのすべてのカラムに `NULL` が指定されます。Primary Key テーブルを使用するテーブルでは、デフォルトでは値カラムに `NULL` が指定され、キーカラムに `NOT NULL` が指定されます。`NULL` の値が元のデータに含まれる場合は、`\N` で表示します。StarRocks はデータのロード時に `\N` を `NULL` として扱います。

**DEFAULT "default_value"**: カラムのデフォルト値。StarRocks にデータをロードする際、カラムにマップされたソースフィールドが空の場合、StarRocks は自動的にカラムにデフォルト値を入力します。次のいずれかの方法でデフォルト値を指定できます。

- **DEFAULT current_timestamp**: 現在の時刻をデフォルト値として使用します。詳細については、[current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md) を参照してください。
- **DEFAULT `<default_value>`**: カラムのデータ型の指定された値をデフォルト値として使用します。たとえば、カラムのデータ型が VARCHAR の場合、`DEFAULT "beijing"` のように、beijing のような VARCHAR 文字列をデフォルト値として指定できます。なお、デフォルト値は次のいずれかの型にすることはできません: ARRAY、BITMAP、JSON、HLL、BOOLEAN。
- **DEFAULT (\<expr\>)**: 指定された関数が返す結果をデフォルト値として使用します。[uuid()](../../sql-functions/utility-functions/uuid.md) および [uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md) 式のみがサポートされます。

**AUTO_INCREMENT**: `AUTO_INCREMENT` カラムを指定します。`AUTO_INCREMENT` カラムのデータ型は BIGINT である必要があります。自動インクリメントされる ID は 1 から始まり、1 ずつ増加します。`AUTO_INCREMENT` カラムに関する詳細については、[AUTO_INCREMENT](../../sql-statements/auto_increment.md) を参照してください。StarRocks は v3.0 以降、`AUTO_INCREMENT` カラムをサポートしています。

**AS generation_expr**: 生成されたカラムとその式を指定します。[生成されたカラム](../generated_columns.md) は、同じ複雑な式を持つクエリの実行を大幅に高速化するために、式の結果を事前計算して格納するために使用できます。StarRocks は v3.1 以降、生成されたカラムをサポートしています。

### index_definition

テーブルを作成する際には、ビットマップインデックスのみ作成できます。パラメータの説明と使用上の注意については、[ビットマップインデックスの作成](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index) を参照してください。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINE タイプ

デフォルト値: `olap`。このパラメータが指定されていない場合、デフォルトで OLAP テーブル（StarRocks ネイティブテーブル）が作成されます。

オプションの値: `mysql`、`elasticsearch`、`hive`、`jdbc`（2.3 以降）、`iceberg`、および `hudi`（2.2 以降）。外部データソースをクエリするための外部テーブルを作成する場合は、`CREATE EXTERNAL TABLE` を指定し、`ENGINE` をこれらの値のいずれかに設定します。詳細については、[外部テーブル](../../../data_source/External_table.md) を参照してください。

**v3.0 以降、Hive、Iceberg、Hudi、および JDBC データソースからデータをクエリするためにカタログを使用することをお勧めします。外部テーブルは非推奨です。詳細については、[Hive カタログ](../../../data_source/catalog/hive_catalog.md)、[Iceberg カタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudi カタログ](../../../data_source/catalog/hudi_catalog.md)、および [JDBC カタログ](../../../data_source/catalog/jdbc_catalog.md) を参照してください。**

**v3.1 以降、StarRocks は Iceberg カタログで Parquet 形式のテーブルを作成できるようになりました。また、[INSERT INTO](../data-manipulation/INSERT.md) を使用してこれらの Parquet 形式の Iceberg テーブルにデータを挿入できます。[Iceberg テーブルの作成](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table) を参照してください。**

**v3.2 以降、StarRocks は Hive カタログで Parquet 形式のテーブルを作成できるようになりました。また、[INSERT INTO](../data-manipulation/INSERT.md) を使用してこれらの Parquet 形式の Hive テーブルにデータを挿入できます。[Hive テーブルの作成](../../../data_source/catalog/hive_catalog.md#create-a-hive-table) を参照してください。**

- MySQL の場合、次のプロパティを指定します:

    ```plaintext
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
        "user" = "your_user_name",
        "password" = "your_password",
        "database" = "database_name",
        "table" = "table_name"
    )
    ```

    注意:

    MySQL の "table_name" は実際のテーブル名を示します。対照的に、CREATE TABLE ステートメントの "table_name" は StarRocks 上のこの MySQL テーブルの名前を示します。これらは異なるか、同じである場合があります。

    StarRocks で MySQL テーブルを作成する目的は、MySQL データベースにアクセスすることです。StarRocks 自体は MySQL のデータを保持または保存しません。

- Elasticsearch の場合、次のプロパティを指定します:

    ```plaintext
    PROPERTIES (

    "hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
    "user" = "root",
    "password" = "root",
    "index" = "tindex",
    "type" = "doc"
    )
    ```

  - `hosts`: Elasticsearch クラスタに接続するために使用する URL。1 つまたは複数の URL を指定できます。
  - `user`: 基本認証が有効になっている Elasticsearch クラスタにログインするための root ユーザーのアカウント。
  - `password`: 前述の root アカウントのパスワード。
  - `index`: Elasticsearch クラスタ内の StarRocks テーブルのインデックス。インデックス名は StarRocks テーブル名と同じです。このパラメータを StarRocks テーブルのエイリアスに設定できます。
  - `type`: インデックスのタイプ。デフォルト値は `doc` です。

- Hive の場合、次のプロパティを指定します:

    ```plaintext
    PROPERTIES (

        "database" = "hive_db_name",
        "table" = "hive_table_name",
        "hive.metastore.uris" = "thrift://127.0.0.1:9083"
    )
    ```

    ここで、database は Hive テーブルの対応するデータベースの名前です。table は Hive テーブルの名前です。hive.metastore.uris はサーバーアドレスです。

- JDBC の場合、次のプロパティを指定します:

    ```plaintext
    PROPERTIES (
    "resource"="jdbc0",
    "table"="dest_tbl"
    )
    ```

    `resource` は JDBC リソース名であり、`table` は宛先テーブルです。

- Iceberg の場合、次のプロパティを指定します:

   ```plaintext
    PROPERTIES (
    "resource" = "iceberg0", 
    "database" = "iceberg", 
    "table" = "iceberg_table"
    )
    ```

    `resource` は Iceberg リソース名です。`database` は Iceberg データベースです。`table` は Iceberg テーブルです。

- Hudi の場合、次のプロパティを指定します:

  ```plaintext
    PROPERTIES (
    "resource" = "hudi0", 
    "database" = "hudi", 
    "table" = "hudi_table" 
    )
    ```

### key_desc

構文:

```SQL
key_type(k1[,k2 ...])
```

データは指定されたキーカラムでシーケンス化され、異なるキータイプに対して異なる属性を持ちます。

- AGGREGATE KEY: キーカラムの同一の内容は、指定された集計タイプに従って値カラムに集計されます。これは、財務諸表や多次元分析などのビジネスシナリオに通常適用されます。
- UNIQUE KEY/PRIMARY KEY: キーカラムの同一の内容は、インポートのシーケンスに従って値カラムに置換されます。これは、キーカラムの追加、削除、変更、およびクエリを行うために使用できます。
- DUPLICATE KEY: キーカラムの同一の内容は、同時に StarRocks に存在するものです。詳細データや集計属性のないデータを格納するために使用できます。**DUPLICATE KEY はデフォルトのタイプです。データはキーカラムに従ってシーケンス化されます。**

> **注意**
>
> AGGREGATE KEY 以外の key_type を使用してテーブルを作成する場合、値カラムには集計タイプを指定する必要はありません。

### COMMENT

テーブルを作成する際にテーブルコメントを追加できます（オプション）。ただし、COMMENT は `key_desc` の後に配置する必要があります。そうしないと、テーブルを作成できません。

v3.1 以降、`ALTER TABLE <table_name> COMMENT = "new table comment"` を実行してテーブルコメントを変更できます。

### partition_desc

パーティションの説明は次のように使用できます。

#### パーティションを動的に作成する

[動的パーティショニング](../../../table_design/dynamic_partitioning.md) は、パーティションのタイム・トゥ・リヴ (TTL) 管理を提供します。StarRocks は、データの新しいパーティションを事前に自動的に作成し、期限切れのパーティションを削除してデータの新鮮さを確保します。この機能を有効にするには、テーブル作成時に Dynamic パーティショニング関連のプロパティを設定できます。

#### 1 つずつパーティションを作成する

**パーティションの上限のみを指定する**

構文:

```sql
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
  PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  [ ,
  PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  , ... ] 
)
```

注意:

パーティショニングに指定されたキーカラムと指定された値範囲を使用してください。

- パーティション名は [A-z0-9_] のみをサポートします。
- Range パーティションのカラムは、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DATE、および DATETIME のみをサポートします。
- パーティションは左閉区間であり、右開区間です。最初のパーティションの左境界は最小値です。
- NULL 値は、最小値を含むパーティションにのみ格納されます。最小値を含むパーティションが削除されると、NULL 値はもはやインポートできません。
- パーティションカラムは単一のカラムまたは複数のカラムにすることができます。パーティション値はデフォルトの最小値です。
- パーティショニングカラムが 1 つだけ指定されている場合、最新のパーティションのパーティショニングカラムの上限に `MAXVALUE` を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

注意事項:

- パーティションは、通常、時間に関連するデータを管理するために使用されます。
- データのバックトラッキングが必要な場合は、必要に応じてパーティションを追加するために最初のパーティションを空にすることを検討してください。

**パーティションの下限と上限を指定する**

構文:

```SQL
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
(
    PARTITION <partition_name1> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1?" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    [,
    PARTITION <partition_name2> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    , ...]
)
```

注意:

- 固定範囲は LESS THAN と同じですが、左右のパーティションをカスタマイズできます。
- 固定範囲は LESS THAN と他の側面では同じです。
- パーティショニングカラムが 1 つだけ指定されている場合、最新のパーティションのパーティショニングカラムの上限に `MAXVALUE` を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### 一括で複数のパーティションを作成する

構文

- パーティショニングカラムが日付型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- パーティショニングカラムが整数型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

説明

一括で複数のパーティションを作成するために、`START()` および `END()` で開始値と終了値を指定し、`EVERY()` で時間単位またはパーティショニングの粒度を指定できます。

- パーティショニングカラムは日付型または整数型である必要があります。
- パーティショニングカラムが日付型の場合、`INTERVAL` キーワードを使用して時間間隔を指定する必要があります。時間単位として hour（v3.0 以降）、day、week、month、year を指定できます。パーティションの命名規則は、動的パーティションと同じです。

詳細については、[データ分散](../../../table_design/Data_distribution.md) を参照してください。

### distribution_desc

StarRocks はハッシュバケットとランダムバケットをサポートしています。バケットを設定しない場合、StarRocks はランダムバケットを使用し、デフォルトでバケットの数を自動的に設定します。

- ランダムバケット（v3.1 以降）

  パーティション内のデータは、特定の列の値に基づいてではなく、ランダムにすべてのバケットに分散されます。また、StarRocks がバケットの数を自動的に決定する場合は、バケットの数を指定する必要はありません。

  - ランダムバケットの場合:

    ```SQL
    DISTRIBUTED BY RANDOM BUCKETS <num>
    ```
  
  ただし、ランダムバケットによって提供されるクエリパフォーマンスは、大量のデータをクエリし、特定の列を頻繁に条件列として使用する場合には理想的ではありません。このシナリオでは、ハッシュバケットを使用することをお勧めします。少数のバケットのみをスキャンおよび計算する必要があるため、クエリパフォーマンスが大幅に向上します。

  **注意事項**
  - ランダムバケットを使用して Duplicate Key テーブルを作成できます。
  - ランダムバケットには [共有結合](../../../using_starrocks/Colocate_join.md) を指定できません。
  - [Spark Load](../../../loading/SparkLoad.md) は、ランダムバケットにデータをロードするために使用できません。
  - StarRocks v2.5.7 以降、テーブルを作成する際にバケットの数を設定する必要はありません。StarRocks はバケットの数を自動的に設定します。このパラメータを設定する場合は、[バケットの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

  詳細については、[ランダムバケット](../../../table_design/Data_distribution.md#random-bucketing-since-v31) を参照してください。

- ハッシュバケット

  構文:

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  パーティションデータは、バケット化された列のハッシュ値とバケットの数に基づいてバケットに分割されます。次の 2 つの要件を満たす列をバケット化列として選択することをお勧めします。

  - ID などの高基数列
  - クエリのフィルタ条件として頻繁に使用される列

  このような列が存在しない場合は、クエリの複雑さに応じてバケット化列を決定できます。

  - クエリが複雑な場合、クエリの複雑さを考慮して、データのバケット間のバランスの取れたデータ分散とクラスタリソースの利用率を向上させるために、高基数列をバケット化列として選択することをお勧めします。
  - クエリが比較的単純な場合、クエリの効率を向上させるために、クエリ条件として頻繁に使用される列をバケット化列として選択することをお勧めします。

  1 つのバケット化列ではパーティションデータを均等にバケットに分散できない場合は、複数のバケット化列（最大 3 つまで）を選択できます。詳細については、[バケット化列の選択](../../../table_design/Data_distribution.md#hash-bucketing) を参照してください。

  **注意事項**:

  - **テーブルを作成する際には、バケット化列を指定する必要があります**。
  - バケット化列の値は更新できません。
  - バケット化列は指定した後に変更できません。
  - StarRocks v2.5.7 以降、テーブルを作成する際にバケットの数を設定する必要はありません。StarRocks はバケットの数を自動的に設定します。このパラメータを設定する場合は、[バケットの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

### ORDER BY

v3.0 以降、Primary Key テーブルでは主キーとソートキーが分離されています。ソートキーは `ORDER BY` キーワードで指定され、任意の列の順列と組み合わせにすることができます。

> **注意**
>
> ソートキーが指定されている場合、プレフィックスインデックスはソートキーに基づいて構築されます。ソートキーが指定されていない場合、プレフィックスインデックスは主キーに基づいて構築されます。

### PROPERTIES

#### 初期ストレージメディア、自動ストレージ冷却時間、レプリカ数を指定する

エンジンタイプが `OLAP` の場合、テーブルを作成する際に初期ストレージメディア (`storage_medium`)、自動ストレージ冷却時間 (`storage_cooldown_time`) または時間間隔 (`storage_cooldown_ttl`)、およびレプリカ数 (`replication_num`) を指定できます。

プロパティが適用される範囲: テーブルにパーティションが 1 つしかない場合、プロパティはテーブルに属します。テーブルが複数のパーティションに分割されている場合、プロパティは各パーティションに属します。また、指定したパーティションごとに異なるプロパティを設定する必要がある場合は、テーブル作成後に [ALTER TABLE ... ADD PARTITION または ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md) を実行できます。

**初期ストレージメディアと自動ストレージ冷却時間を設定する**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`: 初期ストレージメディアです。`SSD` または `HDD` に設定できます。明示的に指定するストレージメディアのタイプが、BE のディスクタイプ（`storage_root_path` の BE 静的パラメータ）と一致していることを確認してください。<br />

    FE の設定項目 `enable_strict_storage_medium_check` が `true` に設定されている場合、システムはテーブルを作成する際に BE のディスクタイプを厳密にチェックします。CREATE TABLE で指定したストレージメディアが BE のディスクタイプと一致しない場合、「Failed to find enough host in all backends with storage medium is SSD|HDD.」というエラーが返され、テーブルの作成に失敗します。`enable_strict_storage_medium_check` が `false` に設定されている場合、システムはこのエラーを無視し、テーブルを強制的に作成します。ただし、データがロードされた後、クラスタのディスクスペースが均等に分散されない可能性があります。<br />

    v2.3.6、v2.4.2、v2.5.1、および v3.0 以降、システムは `storage_medium` が明示的に指定されていない場合、BE のディスクタイプに基づいてストレージメディアを自動的に推論します。<br />

  - システムは次のシナリオでこのパラメータを自動的に SSD に設定します:

    - BE が報告するディスクタイプ (`storage_root_path`) に SSD のみが含まれる場合。
    - BE が報告するディスクタイプ (`storage_root_path`) に SSD と HDD の両方が含まれる場合。なお、v2.3.10、v2.4.5、v2.5.4、および v3.0 以降、この場合、`storage_root_path` に SSD と HDD の両方が含まれ、プロパティ `storage_cooldown_time` が指定されている場合、システムは `storage_medium` を SSD に設定します。

  - システムは次のシナリオでこのパラメータを自動的に HDD に設定します:

    - BE が報告するディスクタイプ (`storage_root_path`) に HDD のみが含まれる場合。
    - v2.3.10、v2.4.5、v2.5.4、および v3.0 以降、BE が報告するディスクタイプ (`storage_root_path`) に SSD と HDD の両方が含まれ、プロパティ `storage_cooldown_time` が指定されていない場合、システムは `storage_medium` を HDD に設定します。

- `storage_cooldown_ttl` または `storage_cooldown_time`: 自動ストレージ冷却時間または時間間隔です。自動ストレージ冷却は、自動的にデータを SSD から HDD に移行することを指します。この機能は、初期ストレージメディアが SSD の場合にのみ有効です。

  **パラメータ**

  - `storage_cooldown_ttl`：このテーブルのパーティションの自動ストレージ冷却の**時間間隔**です。最新のパーティションを SSD に保持し、一定の時間間隔後に古いパーティションを自動的に HDD に冷却する場合は、このパラメータを使用できます。各パーティションの自動ストレージ冷却時間は、このパラメータの値とパーティションの上限時間の合計で計算されます。

  サポートされる値は `<num> YEAR`、`<num> MONTH`、`<num> DAY`、`<num> HOUR` です。`<num>` は非負の整数です。デフォルト値は null で、自動ストレージ冷却は自動的に実行されません。
```SQL
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048),
    v2 SMALLINT SUM DEFAULT "10"
)
ENGINE=olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type"="column",
    "sort_columns"="k1,k2"
);
```
ユーザーのアドレスと最終アクティブ時間などの次元から、リアルタイムでユーザーの行動を分析する必要があるとします。テーブルを作成する際に、`user_id`列を主キーとして定義し、`address`列と`last_active`列の組み合わせをソートキーとして定義することができます。

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

## 参考文献

- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [USE](USE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [DROP TABLE](DROP_TABLE.md)
