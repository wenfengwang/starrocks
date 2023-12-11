---
displayed_sidebar: "Japanese"
---

# テーブルの作成

## 説明

StarRocks に新しいテーブルを作成します。

> **注意**
>
> この操作を行うには、宛先データベースで CREATE TABLE 権限が必要です。

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

通常、カラム名に `__op` または `__row` を先頭に付けることはできません。これらの名前形式は StarRocks で特定の目的のために予約されており、このようなカラムを作成すると未定義の動作を引き起こす可能性があります。このようなカラムを作成する必要がある場合は、FE ダイナミックパラメータ [`allow_system_reserved_names`](../../../administration/Configuration.md#allow_system_reserved_names) を `TRUE` に設定してください。

**col_type**: カラム型。以下のカラム情報が特定されています:

- TINYINT (1 バイト): -2^7 + 1 ～ 2^7 - 1 の範囲。
- SMALLINT (2 バイト): -2^15 + 1 ～ 2^15 - 1 の範囲。
- INT (4 バイト): -2^31 + 1 ～ 2^31 - 1 の範囲。
- BIGINT (8 バイト): -2^63 + 1 ～ 2^63 - 1 の範囲。
- LARGEINT (16 バイト): -2^127 + 1 ～ 2^127 - 1 の範囲。
- FLOAT (4 バイト): 科学的表記をサポートします。
- DOUBLE (8 バイト): 科学的表記をサポートします。
- DECIMAL[(precision, scale)] (16 バイト)

  - デフォルト値: DECIMAL(10, 0)
  - precision: 1 ～ 38
  - scale: 0 ～ precision
  - 整数部: precision - scale

    Scientific notation is not supported.

- DATE (3 バイト): 0000-01-01 ～ 9999-12-31 の範囲。
- DATETIME (8 バイト): 0000-01-01 00:00:00 ～ 9999-12-31 23:59:59 の範囲。
- CHAR[(length)]: 固定長文字列。範囲: 1 ～ 255。デフォルト値: 1。
- VARCHAR[(length)]: 可変長文字列。デフォルト値は 1 です。単位: バイト。StarRocks 2.1 より前のバージョンでは、`length` の値範囲は 1～65533 です。[プレビュー] StarRocks 2.1 および以降のバージョンでは、`length` の値範囲は 1～1048576 です。
- HLL (1～16385 バイト): HLL タイプには長さやデフォルト値の指定は必要ありません。長さはデータ集約に応じてシステムで制御されます。HLL カラムは [hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md), [Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md), および [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) でのみクエリや使用が可能です。
- BITMAP: Bitmap タイプには指定された長さやデフォルト値は必要ありません。符号なし bigint 数のセットを表します。最大要素は 2^64 - 1 までです。

**agg_type**: 集約タイプ。指定されていない場合、このカラムはキーカラムです。
指定された場合、値カラムです。以下の集約タイプがサポートされています:

- SUM, MAX, MIN, REPLACE
- HLL_UNION (HLL タイプのみ)
- BITMAP_UNION (BITMAP のみ)
- REPLACE_IF_NOT_NULL: 取り込まれるデータが非 NULL 値の場合のみ置換されます。NULL 値の場合、StarRocks は元の値を保持します。

> 注意
>
> - 集約タイプが BITMAP_UNION のカラムをインポートする際、元のデータ型は TINYINT、SMALLINT、INT、および BIGINT である必要があります。
> - REPLACE_IF_NOT_NULL カラムに NOT NULL が指定されていた場合、StarRocks はエラー報告をユーザーに送信せずにデータを NULL に変換します。これによりユーザーは選択したカラムをインポートできます。

この集約タイプは、キーのタイプが AGGREGATE KEY である集計テーブルにのみ適用されます。

**NULL | NOT NULL**: カラムが `NULL` であることを許可するかどうか。Duplicate Key、Aggregate、または Unique Key テーブルを使用しているテーブルのすべてのカラムにデフォルトで `NULL` が指定されています。Primary Key テーブルを使用しているテーブルでは、デフォルトで値カラムが `NULL` で指定されていますが、キーカラムでは `NOT NULL` で指定されています。生データに `NULL` 値が含まれている場合、 `\N` で表示してください。StarRocks はデータロード時に `\N` を `NULL` として扱います。

**DEFAULT "default_value"**: カラムのデフォルト値。StarRocks にデータをロードする際、カラムにマップされたソースフィールドが空の場合、StarRocks は自動的にカラムにデフォルト値を埋めます。デフォルト値は以下の方法で指定できます:

- **DEFAULT current_timestamp**: 現在の時間をデフォルト値として使用します。詳細については、[current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md) を参照してください。
- **DEFAULT `<default_value>`**: カラムのデータ型の指定された値をデフォルト値として使用します。たとえば、カラムのデータ型が VARCHAR の場合、`DEFAULT "beijing"` のように VARCHAR 文字列をデフォルト値として指定できます。なお、デフォルト値は次のいずれかであってはなりません: ARRAY、BITMAP、JSON、HLL、BOOLEAN。
- **DEFAULT (\<expr\>)**: 指定された関数によって返された結果をデフォルト値として使用します。[uuid()](../../sql-functions/utility-functions/uuid.md) および [uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md) 式のみがサポートされています。

**AUTO_INCREMENT**: `AUTO_INCREMENT` カラムを指定します。`AUTO_INCREMENT` カラムのデータ型は BIGINT である必要があります。自動生成された ID は 1 から始まり、1 ずつ増加します。`AUTO_INCREMENT` カラムについての詳細は、[AUTO_INCREMENT](../../sql-statements/auto_increment.md) を参照してください。v3.0 以降、StarRocks は `AUTO_INCREMENT` カラムをサポートしています。

**AS generation_expr**: 生成されたカラムとその式を指定します。[生成されたカラム](../generated_columns.md) を使用すると、同じ複雑な式を持つクエリを大幅に高速化できます。v3.1 以降、StarRocks は生成されたカラムをサポートしています。

### index_definition

テーブルを作成する際には、ビットマップインデックスのみを作成できます。パラメータの説明および使用上の注意については、[ビットマップインデックス](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index) を参照してください。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINE タイプ

デフォルト値: `olap`。このパラメータが指定されていない場合、デフォルトで OLAP テーブル (StarRocks 固有テーブル) が作成されます。

オプション値: `mysql`、`elasticsearch`、`hive`、`jdbc` (2.3 以降)、`iceberg`、および `hudi` (2.2 以降)。外部データソースをクエリするための外部テーブルを作成する場合は、`CREATE EXTERNAL TABLE` を指定し、`ENGINE` をこれらの値のいずれかに設定してください。詳細については、[External table](../../../data_source/External_table.md) を参照してください。

**v3.0 以降、Hive、Iceberg、Hudi、JDBC データソースからデータを問い合わせるためにカタログの使用を推奨しています。外部テーブルは非推奨です。詳細については、[Hive catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)、および[JDBC catalog](../../../data_source/catalog/jdbc_catalog.md)を参照してください。**

**v3.1 以降、StarRocks は Iceberg カタログに Parquet 形式のテーブルを作成することをサポートし、これらの Parquet 形式の Iceberg テーブルに対して [INSERT INTO](../data-manipulation/INSERT.md) を使用してデータを挿入できます。[Iceberg table](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table) を参照してください。**

**v3.2 以降、StarRocks は Hive カタログに Parquet 形式のテーブルを作成することをサポートし、これらの Parquet 形式の Hive テーブルに対して [INSERT INTO](../data-manipulation/INSERT.md) を使用してデータを挿入できます。[Hive table](../../../data_source/catalog/hive_catalog.md#create-a-hive-table) を参照してください。**

- MySQL の場合、以下のプロパティが指定されています:

    ```plaintext
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
```plaintext
"user" = "your_user_name",
"password" = "your_password",
"database" = "database_name",
"table" = "table_name"
)
```

ノート：

MySQLの「table_name」は、実際のテーブル名を示す必要があります。一方、CREATE TABLEステートメントの「table_name」はStarRocksのこのMySQLテーブルの名前を示します。これらは異なるか同じかのいずれかであっても構いません。

StarRocksでMySQLテーブルを作成する目的は、MySQLデータベースにアクセスすることです。StarRocks自体はMySQLデータを管理または保存しません。

- Elasticsearchの場合、次のプロパティを指定します：

```plaintext
PROPERTIES (

"hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
"user" = "root",
"password" = "root",
"index" = "tindex",
"type" = "doc"
)
```

  - `hosts`：Elasticsearchクラスタに接続するために使用されるURL。1つ以上のURLを指定できます。
  - `user`：基本認証が有効になっているElasticsearchクラスタにログインするために使用されるrootユーザのアカウントです。
  - `password`：先述のrootアカウントのパスワード。
  - `index`：Elasticsearchクラスタ内のStarRocksテーブルのインデックス。インデックス名はStarRocksテーブルの名前と同じです。このパラメータをStarRocksテーブルのエイリアスに設定することもできます。
  - `type`：インデックスのタイプ。デフォルト値は「doc」です。

- Hiveの場合、次のプロパティを指定します：

```plaintext
PROPERTIES (

    "database" = "hive_db_name",
    "table" = "hive_table_name",
    "hive.metastore.uris" = "thrift://127.0.0.1:9083"
)
```

ここで、databaseはHiveテーブルの対応するデータベースの名前です。TableはHiveテーブルの名前です。hive.metastore.urisはサーバーアドレスです。

- JDBCの場合、次のプロパティを指定します：

```plaintext
PROPERTIES (
"resource"="jdbc0",
"table"="dest_tbl"
)
```

`resource`はJDBCリソース名であり、 `table`は宛先テーブルです。

- Icebergの場合、次のプロパティを指定します：

```plaintext
PROPERTIES (
"resource" = "iceberg0", 
"database" = "iceberg", 
"table" = "iceberg_table"
)
```

`resource`はIcebergリソース名です。`database`はIcebergデータベースです。`table`はIcebergテーブルです。

- Hudiの場合、次のプロパティを指定します：

```plaintext
PROPERTIES (
"resource" = "hudi0", 
"database" = "hudi", 
"table" = "hudi_table" 
)
```

### key_desc

構文：

```SQL
key_type(k1[,k2 ...])
```

データは指定したキー列で並べ替えられ、異なるキータイプごとに異なる属性を持っています。

- AGGREGATE KEY: キー列の同一内容は、指定された集計タイプに従って値列に集約されます。通常、財務諸表や多次元分析などのビジネスシナリオに適用されます。
- UNIQUE KEY/PRIMARY KEY: キー列の同一内容は、インポートシーケンスに従って値列で置換されます。キー列に追加、削除、変更、およびクエリを行うために適用できます。
- DUPLICATE KEY: キー列の同一内容は、同時にStarRocksにも存在します。詳細データや集計属性のないデータを保存するために使用できます。**DUPLICATE KEYはデフォルトのタイプです。データはキー列に従って並べ替えられます。**

> **注意**
>
> AGGREGATE KEYを除く他のkey_typeを使用してテーブルを作成する場合は、値列に集計タイプを指定する必要はありません。

### COMMENT

テーブルを作成する際にテーブルコメントを追加することができますが、オプションです。COMMENTは`key_desc`の後に配置する必要があります。そうしないと、テーブルを作成できません。

v3.1以降、`ALTER TABLE <table_name> COMMENT = "new table comment"`を使用してテーブルコメントを変更できます。

### partition_desc

パーティション記述には、次のような方法があります:

#### 動的にパーティションを作成

[動的パーティション](../../../table_design/dynamic_partitioning.md)はパーティションの有効期限（TTL）管理を提供します。StarRocksは、データの新鮮さを確保するために、自動的に新しいパーティションを事前に作成し、期限切れのパーティションを削除します。この機能を有効にするには、テーブルの作成時に動的パーティションに関連するプロパティを構成できます。

#### 一つずつパーティションを作成

**パーティションの上限のみを指定**

構文:

```sql
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
  PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  [ ,
  PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  , ... ] 
)
```

ノート：

指定されたキーカラムと指定された値範囲を使用してパーティションを行う必要があります。

- パーティション名は[A-z0-9_]のみをサポートしています。
- Rangeパーティション内の列はTINYINT、SMALLINT、INT、BIGINT、LARGEINT、DATE、およびDATETIMEのみをサポートしています。
- パーティションは左が閉じられており、右が開いています。最初のパーティションの左境界は最小値です。
- NULL値は最小値を含むパーティションにのみ保存されます。最小値を含むパーティションが削除されると、もはやNULL値をインポートできません。
- パーティションの列は単一列または複数列にすることができます。パーティション値はデフォルトの最小値です。
- パーティション化列が1つだけ指定されている場合、最新のパーティションのパーティション化列の上限として `MAXVALUE`を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

注意：

- パーティションは、時間に関連するデータを管理するためによく使用されます。
- データのバックトラックが必要な場合は、必要に応じて後でパーティションを追加するために最初のパーティションを空にすることを考慮する必要があります。

**パーティションの下限と上限を指定**

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

ノート：

- 固定範囲はLESS THANよりも柔軟です。左と右のパーティションをカスタマイズできます。
- 固定の範囲は他の点ではLESS THANと同じです。
- パーティション化列が1つだけ指定されている場合、最新のパーティションのパーティション化列の上限として `MAXVALUE`を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### 一括で複数のパーティションを作成

構文

- パーティション化列が日付型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- パーティション化列が整数型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

説明

`START()` と `END()` で開始値や終了値を指定し、`EVERY()`で時間単位やパーティション化の粒度を指定して複数のパーティションを一括で作成できます。

- パーティション化列は日付型または整数型のいずれかです。
- パーティション化列が日付型の場合は、`INTERVAL`キーワードを使用して時間間隔を指定する必要があります。時間単位としては、時間(3.0以降)、日、週、月、年を指定できます。パーティションの命名規則は動的パーティションと同じです。

詳細については、[データ配布](../../../table_design/Data_distribution.md)を参照してください。

### distribution_desc

StarRocksはハッシュバケットとランダムバケットをサポートしています。バケティングを構成しない場合、StarRocksはランダムバケティングを使用し、デフォルトでバケット数を自動的に設定します。

- ランダムバケティング (v3.1以降)
StarRocksは、パーティション内のデータをランダムにすべてのバケットに分散させます。これは特定の列の値に基づくものではありません。そして、StarRocksにバケットの数を自動的に決定させる場合は、バケット構成を指定する必要はありません。バケットの数を手動で指定する場合は、次の構文を使用します：

```SQL
DISTRIBUTED BY RANDOM BUCKETS <num>
```

ただし、クエリパフォーマンスはランダムなバケット分割を使用すると、大量のデータをクエリし、特定の列を頻繁に使用する場合には理想的ではないかもしれません。このような場合は、ハッシュバケットを使用することをお勧めします。ハッシュバケットでは、スキャンおよび計算する必要があるバケットが少数であるため、クエリのパフォーマンスが大幅に向上します。

**注意点**
- ランダムなバケット分割は、重複キーのテーブルを作成する際にのみ使用できます。
- テーブルのランダムなバケット分割のための[共有グループ] (../../../using_starrocks/Colocate_join.md)を指定することはできません。
- [Spark Load] (../../../loading/SparkLoad.md)を使用してテーブルのデータをロードすることはできません。
- StarRocks v2.5.7以降、テーブルを作成する際にはバケットの数を指定する必要はありません。StarRocksは自動的にバケットの数を設定します。このパラメータを設定する場合は、[バケットの数を決定する] (../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

詳細については、[ランダムなバケット分割] (../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

- ハッシュバケッティング

構文：

```SQL
DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
```

パーティション内のデータは、バケット化された列のハッシュ値とバケットの数に基づいてバケットに細分化されます。以下の2つの要件を満たす列をバケット化の列として選択することをお勧めします。

- IDなどの高基数列
- クエリでよく使用される列

そのような列が存在しない場合は、クエリの複雑さに応じてバケット化の列を決定できます。

- クエリが複雑な場合、データ分布を均等にし、クラスタのリソース利用を向上させるため、高基数列をバケット化の列として選択することをお勧めします。
- クエリが比較的単純な場合、クエリ効率を向上させるために、クエリ条件として頻繁に使用される列をバケット化の列として選択することをお勧めします。

1つのバケット化の列でパーティションデータを均等にバケットに分散させることができない場合は、複数のバケット化の列（最大で3つ）を選択することができます。詳細については、[バケット化の列の選択] (../../../table_design/Data_distribution.md#hash-bucketing)を参照してください。

**注意点**：

- **テーブルを作成する際に、バケット化の列を必ず指定する必要があります**。
- バケット化の列の値は更新できません。
- バケット化の列は、指定された後で変更することはできません。
- StarRocks v2.5.7以降、テーブルを作成する際にはバケットの数を指定する必要はありません。StarRocksは自動的にバケットの数を設定します。このパラメータを設定する場合は、[バケットの数を決定する] (../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

### ORDER BY

3.0以降のバージョンでは、プライマリキーテーブルのプライマリキーとソートキーが切り離されています。ソートキーは`ORDER BY`キーワードで指定され、任意の列の順列と組み合わせにすることができます。

> **お知らせ**
>
> ソートキーが指定されている場合、ソートキーに従ってプレフィックスインデックスが構築されます。ソートキーが指定されていない場合、プレフィックスインデックスはプライマリキーに従って構築されます。

### プロパティ

#### 初期ストレージメディア、自動ストレージ冷却時間、レプリカ数を指定

エンジンのタイプが `OLAP` の場合、テーブルを作成する際に、初期ストレージメディア（`storage_medium`）、自動ストレージ冷却時間（`storage_cooldown_time` または `storage_cooldown_ttl`）、およびレプリカ数（`replication_num`）を指定できます。

プロパティが適用される範囲：テーブルが1つのパーティションの場合、プロパティはテーブルに属します。テーブルが複数のパーティションに分割される場合、プロパティはそれぞれのパーティションに属します。および指定されたパーティションに異なるプロパティを設定する必要がある場合は、テーブル作成後に[ALTER TABLE ... ADD PARTITION]または[ALTER TABLE ... MODIFY PARTITION]を実行できます。

**初期ストレージメディアおよび自動ストレージ冷却時間を設定**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`：初期ストレージメディアで、`SSD`または`HDD`に設定できます。明示的に指定したストレージメディアのタイプは、StarRocksクラスタでのBE静的パラメータ`storage_root_path`で指定されたBEディスクのタイプと一致していることを確認してください。<br />

  FEの設定項目`enable_strict_storage_medium_check`が`true`に設定されている場合、テーブルを作成する際にシステムはBEディスクのタイプを厳密にチェックします。CREATE TABLEで指定したストレージメディアがBEディスクのタイプと一致しない場合、「Failed to find enough host in all backends with storage medium is SSD|HDD.」のエラーが返され、テーブルの作成に失敗します。`enable_strict_storage_medium_check`が`false`に設定されている場合、システムはこのエラーを無視してテーブルを強制的に作成します。ただし、データをロードした後、クラスタのディスクスペースが均等に分布されない可能性があります。<br />

  v2.3.6以降、v2.4.2、v2.5.1、およびv3.0以降では、`storage_medium`が明示的に指定されていない場合、システムはBEディスクのタイプに基づいて自動的にストレージメディアを推論します。<br />

  - システムは、次のシナリオでこのパラメータを自動的にSSDに設定します：

    - BE（`storage_root_path`）によって報告されたディスクのタイプがSSDのみである場合。
    - BE（`storage_root_path`）によって報告されたディスクのタイプにSSDとHDDの両方が含まれている場合。但し、v2.3.10、v2.4.5、v2.5.4、およびv3.0以降では、BEがSSDとHDDの両方を含む`storage_root_path`を報告し、プロパティ`storage_cooldown_time`が指定されている場合、システムは`storage_medium`をSSDに設定します。

  - システムは、次のシナリオでこのパラメータを自動的にHDDに設定します：

    - BE（`storage_root_path`）によって報告されたディスクのタイプがHDDのみである場合。
    - v2.3.10、v2.4.5、v2.5.4、およびv3.0以降、BEがSSDとHDDの両方を含む`storage_root_path`を報告し、プロパティ`storage_cooldown_time`が指定されていない場合、システムは`storage_medium`をHDDに設定します。

- `storage_cooldown_ttl`または `storage_cooldown_time`：自動ストレージ冷却時間または時間間隔。自動ストレージ冷却とは、自動的にデータをSSDからHDDに移行する機能です。この機能は、初期ストレージメディアがSSDの場合にのみ有効です。

  **パラメータ**

  - `storage_cooldown_ttl`：このテーブルのパーティションの自動ストレージ冷却の**時間間隔**。最新のパーティションをSSDに保持し、一定の時間間隔後に古いパーティションをHDDに自動的に冷却する必要がある場合は、このパラメータを使用できます。各パーティションの自動ストレージ冷却時間は、このパラメータの値とパーティションの上限時間の値によって計算されます。

  サポートされる値は`<num> YEAR`、`<num> MONTH`、`<num> DAY`、`<num> HOUR`です。`<num>`は非負の整数です。デフォルト値はnullで、自動ストレージ冷却は自動的に実行されません。

  たとえば、テーブルを作成する際にこの値を`"storage_cooldown_ttl"="1 DAY"`と指定し、範囲が`[2023-08-01 00:00:00,2023-08-02 00:00:00)`のパーティション`p20230801`が存在する場合、このパーティションの自動ストレージ冷却時間は`2023-08-02 00:00:00`であり、`2023-08-02 00:00:00 + 1 DAY`です。テーブルを作成する際にこの値を`"storage_cooldown_ttl"="0 DAY"`と指定した場合、このパーティションの自動ストレージ冷却時間は`2023-08-02 00:00:00`です。

  - `storage_cooldown_time`：テーブルがSSDからHDDに冷却される際の自動ストレージ冷却時間（**絶対時間**）。指定した時間は現在の時間よりも後である必要があります。パーティションごとに異なるプロパティを設定する必要がある場合は、[ALTER TABLE ... ADD PARTITION]または[ALTER TABLE ... MODIFY PARTITION]を実行できます。

**使用方法**

- 自動ストレージ冷却に関連するパラメータ間の比較は次のとおりです：
  - `storage_cooldown_ttl`：テーブル内のパーティションの自動ストレージ冷却の時間間隔を指定するテーブルプロパティ。システムはパーティションの上限時間を`このパラメータの値とパーティションの上限時間の値`で自動的に冷却します。そのため、自動ストレージ冷却はパーティションの粒度で実行されるため、柔軟性があります。
- `storage_cooldown_time`: このテーブルの自動ストレージクールダウン時間（**絶対時間**）を指定するテーブルプロパティです。また、テーブル作成後に指定されたパーティションに対して異なるプロパティを設定できます。
- `storage_cooldown_second`: クラスタ内のすべてのテーブルに対して自動ストレージクールダウン待機時間を指定する静的FEパラメータです。

- テーブルプロパティ`storage_cooldown_ttl`または`storage_cooldown_time`が、FE静的パラメータ`storage_cooldown_second`より優先されます。
- これらのパラメータを構成する際には、「`storage_medium = "SSD"`」を指定する必要があります。
- これらのパラメータを構成しないと、自動ストレージクールダウンは自動的に実行されません。
- 各パーティションの自動ストレージクールダウン時間を表示するには、`SHOW PARTITIONS FROM <table_name>`を実行します。

**制限**

- 式およびリストのパーティション分割はサポートされていません。
- パーティション列は日付型である必要があります。
- 複数のパーティション列はサポートされていません。
- プライマリキーテーブルはサポートされていません。

**パーティションごとの各テーブルのレプリカ数を設定**

`replication_num`: パーティションごとの各テーブルのレプリカ数。デフォルト数: `3`。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### カラムにBloomフィルターインデックスを追加

エンジンタイプがOLAPの場合、カラムをBloomフィルターインデックスに採用することができます。

Bloomフィルターインデックスを使用する際の以下の制限が適用されます:

- Duplicate KeyまたはPrimary KeyテーブルのすべてのカラムにBloomフィルターインデックスを作成することができます。AggregateテーブルまたはUnique Keyテーブルの場合、キーカラムにのみBloomフィルターインデックスを作成できます。
- TINYINT、FLOAT、DOUBLE、およびDECIMALのカラムはBloomフィルターインデックスの作成をサポートしていません。
- Bloomフィルターインデックスは、`in`および`=`演算子を含むクエリのパフォーマンスを向上させることができます。例: `Select xxx from table where x in {}` および `Select xxx from table where column = xxx`。このカラムにより離散値が増加すると、より正確なクエリが生成されます。

詳細については、[Bloom filter indexing](../../../using_starrocks/Bloomfilter_index.md)を参照してください。

```SQL
PROPERTIES (
    "bloom_filter_columns" = "k1,k2,k3"
)
```

#### Colocate Joinを使用

Colocate Join属性を使用したい場合は、`properties`で指定します。

```SQL
PROPERTIES (
    "colocate_with"="table1"
)
```

#### 動的パーティションを構成する

動的パーティション属性を使用する場合は、`properties`で指定してください。

```SQL
PROPERTIES (
    "dynamic_partition.enable" = "true|false",
    "dynamic_partition.time_unit" = "DAY|WEEK|MONTH",
    "dynamic_partition.start" = "${integer_value}",
    "dynamic_partition.end" = "${integer_value}",
    "dynamic_partition.prefix" = "${string_value}",
    "dynamic_partition.buckets" = "${integer_value}"
```

**`PROPERTIES`**

| パラメータ                   | 必須     | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| dynamic_partition.enable    | いいえ       | 動的パーティション分割を有効にするかどうか。有効な値: `TRUE` および `FALSE`。デフォルト値: `TRUE`。 |
| dynamic_partition.time_unit | はい      | 動的に作成されたパーティションの時間単位。必須のパラメータです。有効な値: `DAY`、`WEEK`、`MONTH`。時間単位は、動的に作成されたパーティションの接尾辞の形式を決定します。<br/>  - 値が`DAY`の場合、動的に作成されたパーティションの接尾辞の形式は`yyyyMMdd`です。例: `20200321` のパーティション名接尾辞。<br/>  - 値が`WEEK`の場合、動的に作成されたパーティションの接尾辞の形式は`yyyy_ww`です。例: 2020年の13週目の`2020_13`。<br/>  - 値が`MONTH`の場合、動的に作成されたパーティションの接尾辞の形式は`yyyyMM`です。例: `202003`。 |
| dynamic_partition.start     | いいえ       | 動的パーティション分割の開始オフセット。このパラメータの値は負の整数である必要があります。`dynamic_partition.time_unit`によって現在の日、週、または月に基づいて削除されるこのオフセットより前のパーティションが削除されます。デフォルト値は`Integer.MIN_VALUE`であり、つまり-2147483648であり、過去のパーティションが削除されません。 |
| dynamic_partition.end       | はい      | 動的パーティション分割の終了オフセット。このパラメータの値は正の整数である必要があります。現在の日、週、または月から終了オフセットまでのパーティションが事前に作成されます。 |
| dynamic_partition.prefix    | いいえ       | 動的パーティションの名前に追加される接頭語。デフォルト値: `p`。 |
| dynamic_partition.buckets   | いいえ       | 動的パーティションごとのバケット数。デフォルト値は、`BUCKETS`で決まるバケット数と同じです。StarRocksによって自動的に設定されます。

#### ランダムバケティングで構成されたテーブルのバケットサイズを指定

v3.2以降、ランダムバケティングで構成されたテーブルに対し、`PROPERTIES`の`bucket_size`パラメータを使用してバケットサイズを指定できます。デフォルトサイズは`1024 * 1024 * 1024 B`（1 GB）で、最大サイズは4 GBです。

```sql
PROPERTIES (
    "bucket_size" = "3221225472"
)
```

#### データ圧縮アルゴリズムを設定

テーブルのデータ圧縮アルゴリズムを指定するには、テーブルを作成する際に`compression`プロパティを追加します。

`compression`の有効な値は次のとおりです:

- `LZ4`: LZ4アルゴリズム。
- `ZSTD`: Zstandardアルゴリズム。
- `ZLIB`: zlibアルゴリズム。
- `SNAPPY`: Snappyアルゴリズム。

適切なデータ圧縮アルゴリズムの選択方法についての詳細は、[Data compression](../../../table_design/data_compression.md)を参照してください。

#### データロード用の書き込みクォーラムを設定

StarRocksクラスタに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定し、つまりStarRocksがデータのロードタスクを完了するために必要なレプリカの数を指定できます。テーブルを作成する際に、`write_quorum`プロパティを追加して書き込みクォーラムを指定できます。このプロパティはv2.5からサポートされています。

`write_quorum`の有効な値は次のとおりです:

- `MAJORITY`: デフォルト値。データレプリカの**大多数**がロード成功を返すと、StarRocksはロードタスクを成功とみなします。それ以外の場合、StarRocksはロードタスクを失敗とみなします。
- `ONE`: データレプリカの**いずれか1つ**がロード成功を返すと、StarRocksはロードタスクを成功とみなします。それ以外の場合、StarRocksはロードタスクを失敗とみなします。
- `ALL`: データレプリカの**すべて**がロード成功を返すと、StarRocksはロードタスクを成功とみなします。それ以外の場合、StarRocksはロードタスクを失敗とみなします。

> **注意**
>
> - ロード用の低い書き込みクォーラムを設定すると、データのアクセス不能およびデータの損失のリスクが増加します。たとえば、2つのレプリカからなるStarRocksクラスタに1つの書き込みクォーラムでデータをロードし、データが1つのレプリカにのみ正常にロードされた場合は、StarRocksはロードタスクが成功したと判断します。しかし、データの生存しているレプリカは一つだけです。ロードされたデータのタブレットを保存するサーバーがダウンした場合、これらのタブレットのデータはアクセスできなくなります。また、サーバーのディスクが壊れた場合、データが消失します。
> - StarRocksは、すべてのデータレプリカがステータスを返した後にのみロードタスクの状態を返します。データレプリカのいずれかがロードステータス不明の場合、StarRocksはロードタスクのステータスを返しません。レプリカでは、ロードのタイムアウトもロード失敗とみなされます。

#### レプリカ間でのデータ書き込みおよび複製モードを指定

StarRocksクラスタに複数のデータレプリカがある場合、`PROPERTIES`内の`replicated_storage`パラメータを使用して、レプリカ間のデータ書き込みおよび複製モードを構成できます。

- `true`（v3.0以降のデフォルト）は「シングルリーダーレプリケーション」を示し、つまりデータはプライマリレプリカにのみ書き込まれます。その他のレプリカはプライマリレプリカからデータを同期します。このモードは複数のレプリカへのデータ書き込みによって引き起こされるCPUコストを大幅に削減します。これはv2.5からサポートされています。
- `false`（v2.5のデフォルト）は「リーダーレスレプリケーション」を示し、つまりプライマリとセカンダリの区別なく複数のレプリカに直接データが書き込まれます。CPUコストはレプリカ数によって増加します。

ほとんどの場合、デフォルト値を使用すると、より優れたデータ書き込みパフォーマンスが得られます。レプリカ間のデータ書き込みおよび複製モードを変更したい場合は、ALTER TABLEコマンドを実行してください。例:

```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
```

#### 一括でロールアップを作成

テーブルを作成する際に、一括でロールアップを作成できます。

構文:

```SQL
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...)
```

#### View Delta Joinクエリの再構築においてUnique Key制約およびForeign Key制約を定義

View Delta Joinシナリオでクエリの再構築を有効にするには、Delta Joinで結合されるテーブルに対して、Unique Key制約`unique_constraints`およびForeign Key制約`foreign_key_constraints`を定義する必要があります。詳細については、[Asynchronous materialized view - Rewrite queries in View Delta Join scenario](../../../using_starrocks/query_rewrite_with_materialized_views.md#query-delta-join-rewrite)をご覧ください。

```SQL
PROPERTIES (
    "unique_constraints" = "<unique_key>[, ...]",
    "foreign_key_constraints" = "
    (<child_column>[, ...]) 
    REFERENCES 
    [catalog_name].[database_name].<parent_table_name>(<parent_column>[, ...])
    [;...]

- `child_column`: テーブルの外部キーです。複数の `child_column` を定義できます。
- `catalog_name`: 結合対象のテーブルが存在するカタログの名前です。このパラメータが指定されていない場合はデフォルトのカタログが使用されます。
- `database_name`: 結合対象のテーブルが存在するデータベースの名前です。このパラメータが指定されていない場合は現在のデータベースが使用されます。
- `parent_table_name`: 結合するテーブルの名前です。
- `parent_column`: 結合する列です。それらは対応するテーブルの主キーまたは一意キーでなければなりません。

> **注意**
>
> - `unique_constraints` および `foreign_key_constraints` はクエリの書き換えにのみ使用されます。テーブルにデータをロードする際、外部キー制約のチェックは保証されません。テーブルにロードされるデータが制約を満たしていることを確認する必要があります。
> - 主キーがあるテーブルの主キーまたは一意キーが、デフォルトで対応する `unique_constraints` です。手動で設定する必要はありません。
> - テーブルの `foreign_key_constraints` 内の `child_column` は、他のテーブルの `unique_constraints` の `unique_key` を参照する必要があります。
> - `child_column` と `parent_column` の数は一致していなければなりません。
> - `child_column` と対応する `parent_column` のデータ型は一致していなければなりません。

#### StarRocks Shared-data クラスタ向けのクラウドネイティブ テーブルを作成する

[StarRocks Shared-data クラスタを使用する](../../../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster) には、以下のプロパティを持つクラウドネイティブ テーブルを作成する必要があります：

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

- `datacache.enable`: ローカルディスク キャッシュを有効にするかどうか。デフォルト: `true`。

  - このプロパティを `true` に設定すると、ロードするデータはオブジェクトストレージとローカルディスクの両方に同時に書き込まれます（クエリの高速化のためのキャッシュとして）。 
  - このプロパティを `false` に設定すると、データはオブジェクトストレージにのみロードされます。

  > **注意**
  >
  > ローカルディスク キャッシュを有効にするには、BE 構成項目 `storage_root_path` でディスクのディレクトリを指定する必要があります。詳細については、[BE Configuration items](../../../administration/Configuration.md#be-configuration-items) を参照してください。

- `datacache.partition_duration`: ホットデータの有効期間。ローカルディスク キャッシュが有効になっている場合、すべてのデータはキャッシュにロードされます。 キャッシュが満杯になると、StarRocks はキャッシュから最近使用されていないデータを削除します。クエリが削除されたデータをスキャンする必要があるとき、StarRocks はデータが有効期間内にあるかどうかをチェックします。データが有効期間内であれば、StarRocks はデータを再度キャッシュにロードします。データが有効期間内でない場合、StarRocks はデータをキャッシュにロードしません。このプロパティは、次の単位で指定できる文字列値です： `YEAR`, `MONTH`, `DAY`, `HOUR` など、たとえば `7 DAY` や `12 HOUR` 。指定されていない場合、すべてのデータはホットデータとしてキャッシュされます。

  > **注意**
  >
  > このプロパティは、`datacache.enable` が `true` に設定されている場合のみ使用できます。

- `enable_async_write_back`: データを非同期でオブジェクトストレージに書き込むかどうか。デフォルト: `false`。

  - このプロパティを `true` に設定すると、ロードタスクはデータをローカルディスク キャッシュに書き込むとすぐに成功を返し、データを非同期でオブジェクトストレージに書き込みます。これにより、ロードのパフォーマンスが向上しますが、潜在的なシステム障害時のデータ信頼性が低下します。
  - このプロパティを `false` に設定すると、ロードタスクはデータをオブジェクトストレージとローカルディスク キャッシュの両方に書き込んだ後にのみ成功を返します。これにより、より高い可用性が確保されますが、ロードのパフォーマンスが低下します。

#### 高速なスキーマ進化を設定する

`fast_schema_evolution`: テーブルの高速スキーマ進化を有効にするかどうか。有効な値は `TRUE`（デフォルト）または `FALSE` です。高速なスキーマ進化を有効にすると、列を追加または削除する際のスキーマ変更の速度が向上し、リソースの使用量が削減されます。現在、このプロパティはテーブル作成時にのみ有効にでき、テーブル作成後に [ALTER TABLE](../../sql-statements/data-definition/ALTER_TABLE.md) を使用して変更することはできません。このパラメータは v3.2.0 以降でサポートされています。

  > **注意**
  >
  > - StarRocks Shared-data クラスタではこのプロパティはサポートされていません。
  > - StarRocks クラスタ内で高速スキーマ進化を無効にするなど、クラスタレベルで高速スキーマ進化を構成する必要がある場合は、FE の動的パラメータ [`fast_schema_evolution`](../../../administration/Configuration.md#fast_schema_evolution) を設定してください。

## 例

### ハッシュ バケッティングおよび列ストレージを使用する集約テーブルを作成する

```SQL
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

### 集約テーブルを作成し、ストレージ媒体と冷却時間を設定する

```SQL
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT SUM DEFAULT "10"
)
ENGINE=olap
UNIQUE KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type"="column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

または

```SQL
CREATE TABLE example_db.table_hash
(
    k1 BIGINT,
    k2 LARGEINT,
    v1 VARCHAR(2048) REPLACE,
    v2 SMALLINT SUM DEFAULT "10"
)
ENGINE=olap
PRIMARY KEY(k1, k2)
DISTRIBUTED BY HASH (k1, k2)
PROPERTIES(
    "storage_type"="column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

### レンジパーティション、ハッシュ バケッティング、列ベースのストレージを使用し、ストレージ媒体と冷却時間を設定した重複キー テーブルを作成する

より小さい

```SQL
CREATE TABLE example_db.table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE=olap
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
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

注意：

このステートメントは、3 つのデータパーティションを作成します:

```SQL
( {    MIN     },   {"2014-01-01"} )
[ {"2014-01-01"},   {"2014-06-01"} )
[ {"2014-06-01"},   {"2014-12-01"} )
```

これらの範囲外のデータはロードされません。

固定範囲

```SQL
CREATE TABLE table_range
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE=olap
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

### MySQL 外部テーブルを作成する

```SQL
CREATE EXTERNAL TABLE example_db.table_mysql
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    k4 VARCHAR(2048),
    k5 DATETIME
)
ENGINE=mysql
PROPERTIES
(
    "host" = "127.0.0.1",
    "port" = "8239",
    "user" = "mysql_user",
    "password" = "mysql_passwd",
    "database" = "mysql_db_test",
    "table" = "mysql_table_test"
)
```
```SQL
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 HLL HLL_UNION,
    v2 HLL HLL_UNION
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

```SQL
CREATE TABLE example_db.example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 BITMAP BITMAP_UNION,
    v2 BITMAP BITMAP_UNION
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

```SQL
CREATE TABLE `t1` 
(
     `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) 
ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES 
(
    "colocate_with" = "t1"
);

CREATE TABLE `t2` 
(
    `id` int(11) COMMENT "",
    `value` varchar(8) COMMENT ""
) 
ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES 
(
    "colocate_with" = "t1"
);
```

```SQL
CREATE TABLE example_db.table_hash
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM,
    INDEX k1_idx (k1) USING BITMAP COMMENT 'xxxxxx'
)
ENGINE=olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1)
PROPERTIES ("storage_type"="column");
```

```SQL
CREATE TABLE example_db.dynamic_partition
(
    k1 DATE,
    k2 INT,
    k3 SMALLINT,
    v1 VARCHAR(2048),
    v2 DATETIME DEFAULT "2014-02-04 15:36:00"
)
ENGINE=olap
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
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "10"
);
```

```SQL
CREATE TABLE site_access (
    datekey INT,
    site_id INT,
    city_code SMALLINT,
    user_name VARCHAR(32),
    pv BIGINT DEFAULT '0'
)
ENGINE=olap
DUPLICATE KEY(datekey, site_id, city_code, user_name)
PARTITION BY RANGE (datekey) (START ("1") END ("5") EVERY (1)
)
DISTRIBUTED BY HASH(site_id)
PROPERTIES ("replication_num" = "3");
```

```SQL
CREATE EXTERNAL TABLE example_db.table_hive
(
    k1 TINYINT,
    k2 VARCHAR(50),
    v INT
)
ENGINE=hive
PROPERTIES
(
    "resource" = "hive0",
    "database" = "hive_db_name",
    "table" = "hive_table_name"
);
```

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

## リファレンス

- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [USE](USE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [DROP TABLE](DROP_TABLE.md)