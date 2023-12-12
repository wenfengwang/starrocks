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

### column_definition

構文：

```SQL
col_name col_type [agg_type] [NULL | NOT NULL] [DEFAULT "default_value"] [AUTO_INCREMENT] [AS generation_expr]
```

**col_name**: カラム名。

通常、StarRocksでは`__op`や`__row`で始まるカラムを作成することはできません。なぜなら、これらの名前形式は特別な目的のために予約されており、そのようなカラムを作成すると未定義の動作が発生する可能性があるからです。もし、そのようなカラムを作成する必要がある場合は、FE ダイナミックパラメータ [`allow_system_reserved_names`](../../../administration/Configuration.md#allow_system_reserved_names) を `TRUE` に設定してください。

**col_type**: カラムのタイプ。以下のカラム情報を参照してください。

- TINYINT (1 バイト)：-2^7 + 1 から 2^7 - 1 までの範囲。
- SMALLINT (2 バイト)：-2^15 + 1 から 2^15 - 1 までの範囲。
- INT (4 バイト)：-2^31 + 1 から 2^31 - 1 までの範囲。
- BIGINT (8 バイト)：-2^63 + 1 から 2^63 - 1 までの範囲。
- LARGEINT (16 バイト)：-2^127 + 1 から 2^127 - 1 までの範囲。
- FLOAT (4 バイト)：科学的記数法をサポート。
- DOUBLE (8 バイト)：科学的記数法をサポート。
- DECIMAL[(precision, scale)] (16 バイト)

  - デフォルト値：DECIMAL(10, 0)
  - precision: 1 ~ 38
  - scale: 0 ~ precision
  - 整数部：precision - scale

    科学的記数法はサポートされていません。

- DATE (3 バイト)：0000-01-01 から 9999-12-31 までの範囲。
- DATETIME (8 バイト)：0000-01-01 00:00:00 から 9999-12-31 23:59:59 までの範囲。
- CHAR[(length)]：固定長の文字列。範囲: 1 ~ 255。デフォルト値: 1。
- VARCHAR[(length)]：可変長の文字列。デフォルト値は 1 です。バージョン 2.1 以前では、`length` の値範囲は 1～65533 です。[プレビュー] StarRocks 2.1 以降のバージョンでは、`length` の値範囲は 1～1048576 です。
- HLL (1～16385 バイト)：HLL タイプの場合、長さやデフォルト値を指定する必要はありません。長さはデータ集計に応じてシステム内で制御されます。HLL カラムは [hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[Hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md)、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) によってのみクエリや使用が可能です。
- BITMAP: Bitmap タイプは長さやデフォルト値を指定する必要はありません。符号なし bigint 数のセットを表します。最大の要素は 2^64 - 1 まで可能です。

**agg_type**: 集計タイプ。指定されていない場合はキーカラムです。指定された場合は値カラムです。サポートされている集計タイプは以下の通りです。

- SUM、MAX、MIN、REPLACE
- HLL_UNION (HLL タイプのみ)
- BITMAP_UNION (BITMAP のみ)
- REPLACE_IF_NOT_NULL: インポートされたデータは、非 NULL 値の場合のみ置き換えられます。NULL 値の場合、StarRocks は元の値を保持します。

> 注意
>
> - BITMAP_UNION タイプの列をインポートする場合、元のデータ型は TINYINT、SMALLINT、INT、BIGINT である必要があります。
> - REPLACE_IF_NOT_NULL カラムに NOT NULL が指定されている場合、StarRocks はユーザーにエラー報告を送信せずに、データを NULL に変換します。これにより、ユーザーは選択した列をインポートできます。

この集計タイプは、AGGREGATE KEY タイプの Aggregate テーブルにのみ適用されます。

**NULL | NOT NULL**: カラムが `NULL` であることを許可するかどうか。デフォルトでは、Duplicate Key、Aggregate、Unique Key テーブルを使用するテーブルのすべてのカラムには `NULL` が指定されます。Primary Key テーブルを使用するテーブルでは、値カラムにはデフォルトで `NULL` が指定されており、キーカラムにはデフォルトで `NOT NULL` が指定されています。`NULL` の値が元のデータに含まれている場合は、`\N` を使用して表します。StarRocks はデータのロード中に `\N` を `NULL` として扱います。

**DEFAULT "default_value"**: カラムのデフォルト値。StarRocks にデータをロードする際、もしソースフィールドがカラムにマップされた場合、StarRocks は自動的にカラムにデフォルト値を入力します。以下のいずれかの方法でデフォルト値を指定できます。

- **DEFAULT current_timestamp**: 現在時刻をデフォルト値として使用します。詳細は、 [current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md) を参照してください。
- **DEFAULT `<default_value>`**: カラムのデータ型の指定値をデフォルト値として使用します。たとえば、カラムのデータ型が VARCHAR の場合、北京などの VARCHAR 文字列を `DEFAULT "beijing"` として指定できます。デフォルト値は次のいずれかのタイプであってはなりません: ARRAY、BITMAP、JSON、HLL、BOOLEAN。
- **DEFAULT (\<expr\>)**: 指定された関数によって返された結果をデフォルト値として使用します。[uuid()](../../sql-functions/utility-functions/uuid.md) と [uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md) 式のみがサポートされています。

**AUTO_INCREMENT**: `AUTO_INCREMENT` カラムを指定します。`AUTO_INCREMENT` カラムのデータ型は BIGINT でなければなりません。自動インクリメントされる ID は 1 から始まり、ステップは 1 で増加します。`AUTO_INCREMENT` カラムについての詳細は、 [AUTO_INCREMENT](../../sql-statements/auto_increment.md) を参照してください。v3.0 以降、StarRocks は `AUTO_INCREMENT` カラムをサポートしています。

**AS generation_expr**: 生成された列とその式を指定します。[生成された列](../generated_columns.md) は、同じ複雑な式を持つクエリを大幅に加速するために、式の結果を事前計算して保存するために使用されます。v3.1 以降、StarRocks は生成された列をサポートしています。

### index_definition

テーブルを作成する際には、ビットマップインデックスのみを作成できます。パラメータの詳細や使用上の注意については、[ビットマップインデックス](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index) を参照してください。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINE タイプ

デフォルト値: `olap`。このパラメータが指定されていない場合、デフォルトで OLAP テーブル (StarRocks ネイティブテーブル) が作成されます。

オプションの値: `mysql`、`elasticsearch`、`hive`、`jdbc` (v2.3 以降)、`iceberg`、`hudi` (v2.2 以降)。外部データソースをクエリするための外部テーブルを作成する場合は、`CREATE EXTERNAL TABLE` を指定し、`ENGINE` をこれらの値のいずれかに設定してください。詳細については、[外部テーブル](../../../data_source/External_table.md) を参照してください。

**v3.0 以降、Hive、Iceberg、Hudi、JDBC のデータソースからデータをクエリするために、カタログの使用を推奨します。外部テーブルは非推奨です。詳細については、[Hive catalog](../../../data_source/catalog/hive_catalog.md)、[Iceberg catalog](../../../data_source/catalog/iceberg_catalog.md)、[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)、[JDBC catalog](../../../data_source/catalog/jdbc_catalog.md) を参照してください。**

**v3.1 以降、StarRocks は Iceberg カタログで Parquet 形式のテーブルを作成し、[INSERT INTO](../data-manipulation/INSERT.md) を使用してこれらの Parquet 形式の Iceberg テーブルにデータを挿入できます。[Iceberg table を作成する](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table) を参照してください。**

**v3.2 以降、StarRocks は Hive カタログで Parquet 形式のテーブルを作成し、[INSERT INTO](../data-manipulation/INSERT.md) を使用してこれらの Parquet 形式の Hive テーブルにデータを挿入できます。[Hive table を作成する](../../../data_source/catalog/hive_catalog.md#create-a-hive-table) を参照してください。**

- MySQL の場合、以下のプロパティを指定してください：

    ```plaintext
    PROPERTIES (
        "host" = "mysql_server_host",
        "port" = "mysql_server_port",
```
```plaintext
"user" = "your_user_name",
"password" = "your_password",
"database" = "database_name",
"table" = "table_name"
)

注意：

在MySQL中，"table_name" 应指示真实表名。在创建表语句中的 "table_name" 表示StarRocks中此mysql表的名称。它们可以是不同的，也可以相同。

在StarRocks创建MySQL表的目的是为了访问MySQL数据库。StarRocks本身不维护或存储任何MySQL数据。

- 对于Elasticsearch，指定以下属性：

```plaintext
PROPERTIES (

"hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
"user" = "root",
"password" = "root",
"index" = "tindex",
"type" = "doc"
)
```

  - `hosts`：用于连接Elasticsearch集群的URL。您可以指定一个或多个URL。
  - `user`：用于登录到已启用基本身份验证的Elasticsearch集群的根用户帐户。
  - `password`：前述根帐户的密码。
  - `index`：Elasticsearch集群中StarRocks表的索引。索引名称与StarRocks表名相同。您可以将此参数设置为StarRocks表的别名。
  - `type`：索引类型。默认值为 `doc`。

- 对于Hive，指定以下属性：

```plaintext
PROPERTIES (

    "database" = "hive_db_name",
    "table" = "hive_table_name",
    "hive.metastore.uris" = "thrift://127.0.0.1:9083"
)
```

这里，database是Hive表中对应数据库的名称。Table是Hive表的名称。hive.metastore.uris是服务器地址。

- 对于JDBC，指定以下属性：

```plaintext
PROPERTIES (
"resource"="jdbc0",
"table"="dest_tbl"
)
```

`resource`是JDBC资源名称，`table`是目标表。

- 对于Iceberg，指定以下属性：

```plaintext
PROPERTIES (
"resource" = "iceberg0", 
"database" = "iceberg", 
"table" = "iceberg_table"
)
```

`resource`是Iceberg资源名称。`database`是Iceberg数据库。`table`是Iceberg表。

- 对于Hudi，指定以下属性：

```plaintext
PROPERTIES (
"resource" = "hudi0", 
"database" = "hudi", 
"table" = "hudi_table" 
)
```

### key_desc

语法：

```SQL
key_type(k1[,k2 ...])
```

数据按指定的键列进行排序，并具有不同的键类型的不同属性：

- 聚合键: 键列中相同的内容将根据指定的聚合类型聚合到值列中。通常适用于财务报表和多维分析等业务场景。
- 唯一键/主键: 键列中相同的内容将根据导入顺序替换值列中的内容。可用于对键列进行添加、删除、修改和查询操作。
- 重复键: 键列中存在相同的内容，同时也存在于StarRocks中。可用于存储详细数据或无聚合属性的数据。**重复键是默认类型。数据将根据键列进行排序。**

> **注意**
>
> 当使用其他 key_type 创建表时，值列无需指定聚合类型，除了聚合键。

### COMMENT

您可以在创建表时添加表注释，可选。请注意，COMMENT 必须放置在 `key_desc` 之后。否则，无法创建表。

从v3.1开始，您可以使用 `ALTER TABLE <table_name> COMMENT = "new table comment"` 修改表注释。

### partition_desc

分区描述可通过以下方式使用：

#### 动态创建分区

[动态分区](../../../table_design/dynamic_partitioning.md) 为分区提供了生存时间（TTL）管理。StarRocks会提前自动创建新分区并删除过期分区，以确保数据新鲜性。要启用此功能，可以在表创建时配置与动态分区相关的属性。

#### 逐一创建分区

**仅指定分区的上限**

语法：

```sql
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
  PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  [ ,
  PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  , ... ] 
)
```

注意：

请使用指定的键列和指定的值范围进行分区。

- 分区名称仅支持 [A-z0-9_]
- 范围分区中的列仅支持以下类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DATE 和 DATETIME。
- 分区为左闭右开。第一分区的左边界值是最小值。
- NULL 值仅存储在包含最小值的分区中。删除包含最小值的分区后，将不能再导入 NULL 值。
- 分区列可以是单列也可以多列，分区值为默认最小值。
- 当仅指定一个列作为分区列时，您可以将 `MAXVALUE` 设置为最近分区的分区列的上限值。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

请注意：

- 分区常用于管理与时间相关的数据。
- 当需要数据回溯时，您可能希望考虑在需要时清空第一个分区以添加后续分区。

**同时指定分区的上限和下限**

语法：

```SQL
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
(
    PARTITION <partition_name1> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1?" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    [,
    PARTITION <partition_name2> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    , ...]
)
```

注意：

- 固定范围比LESS THAN更灵活。您可以自定义左右分区。
- 固定范围在其他方面与LESS THAN相同。
- 当仅指定一个列作为分区列时，您可以将 `MAXVALUE` 设置为最近分区的分区列的上限值。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### 批量创建多个分区

语法

- 如果分区列是日期类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- 如果分区列是整数类型。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

描述

您可以在 `START()` 和 `END()` 指定开始和结束值，并在 `EVERY()` 指定时间单位或分区粒度，以批量创建多个分区。

- 分区列既可以是日期类型也可以是整数类型。
- 如果分区列是日期类型，您需要使用 `INTERVAL` 关键字指定时间间隔。您可以将时间单位指定为小时（自v3.0起）、天、周、月或年。分区命名约定与动态分区相同。

有关更多信息，请参阅[数据分布](../../../table_design/Data_distribution.md)。

### distribution_desc

StarRocks支持哈希分桶和随机分桶。如果未配置分桶，则StarRocks使用随机分桶，并默认自动设置分桶数量。

- 随机分桶（自v3.1起）
```
StarRocksでは、パーティションのデータはランダムにすべてのバケツに分散されます。これは特定の列値に基づいているわけではありません。StarRocksにバケツの数を自動的に決定させたい場合は、バケツの構成を指定する必要はありません。バケツの数を手動で指定する場合、構文は次のとおりです。

```SQL
DISTRIBUTED BY RANDOM BUCKETS <num>
```

ただし、ランダムなバケツ分けによるクエリパフォーマンスは、大量のデータをクエリし特定の列を頻繁に使用する場合には理想的とは言えません。このようなシナリオでは、ハッシュバケツ分けを使用することをお勧めします。なぜなら、スキャンおよび計算する必要があるバケツ数が少ないため、クエリパフォーマンスが大幅に向上するからです。

**注意点**
- ランダムなバケツ分けを使用して重複キーのテーブルを作成できます。
- ランダムにバケツ分けされたテーブルには[コロケーショングループ](../../../using_starrocks/Colocate_join.md)を指定できません。
- [Spark Load](../../../loading/SparkLoad.md)は、ランダムなバケツ分けされたテーブルにデータをロードするために使用できません。
- StarRocks v2.5.7以降、テーブルを作成するときにバケツの数を設定する必要はありません。StarRocksは自動的にバケツの数を設定します。このパラメータを設定する必要がある場合は、[バケツの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

詳細については、[ランダムバケツ分け](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

- ハッシュバケツ分け

  構文:

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  パーティション内のデータは、バケツの列のハッシュ値とバケツの数に基づいてバケツに細分化できます。次の2つの要件を満たす列をバケツの列として選択することをお勧めします。

  - IDなどの高カーディナリティ列
  - クエリで頻繁に使用される列

  そのような列が存在しない場合は、クエリの複雑さに応じてバケツの列を決定できます。

  - クエリが複雑な場合、バケツの列として高カーディナリティ列を選択し、バケツ間でデータ分布を均等にし、クラスタリソースの利用を改善します。
  - クエリが比較的単純な場合、クエリ効率を改善するためにクエリ条件としてよく使用される列をバケツの列として選択することをお勧めします。

  1つのバケツの列ではパーティションデータを均等に分散できない場合は、複数のバケツの列（最大3つ)を選択できます。詳細については、[バケツの列を選択](../../../table_design/Data_distribution.md#hash-bucketing)を参照してください。

  **注意点**:

  - **テーブルを作成する際は、そのバケツの列を指定する必要があります**。
  - バケツの列の値は更新できません。
  - バケツの列は、指定した後には変更できません。
  - StarRocks v2.5.7以降、テーブルを作成するときにバケツの数を設定する必要はありません。StarRocksは自動的にバケツの数を設定します。このパラメータを設定する必要がある場合は、[バケツの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

### ORDER BY

バージョン3.0以降、プライマリキーとソートキーはプライマリキーテーブルで切り離されます。ソートキーは`ORDER BY`キーワードで指定され、任意の列の順列と組み合わせにすることができます。

> **注意**
>
> ソートキーが指定された場合、プレフィックスインデックスはソートキーに従って構築されます。ソートキーが指定されていない場合、プレフィックスインデックスはプライマリキーに従って構築されます。

### PROPERTIES

#### 初期保管メディア、自動保管冷却時間、レプリカ数を指定

エンジンタイプが`OLAP`の場合、テーブルを作成する際に初期保管メディア（`storage_medium`）、自動保管冷却時間（`storage_cooldown_time`または時間間隔（`storage_cooldown_ttl`）、およびレプリカ数（`replication_num`）を指定できます。

プロパティが適用される範囲: テーブルが1つのパーティションしかない場合、プロパティはテーブルに属します。テーブルが複数のパーティションに分割されている場合、プロパティはそれぞれのパーティションに属します。また、特定のパーティションに異なるプロパティを構成する必要がある場合は、テーブル作成後に[ALTER TABLE ... ADD PARTITIONまたはALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)を実行できます。

**初期保管メディアと自動保管冷却時間を指定**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`: 初期の保管メディア。`SSD`または`HDD`に設定できます。明示的に指定した保管メディアの種類は、StarRocksクラスタのBE静的パラメータ`storage_root_path`で指定されたBEディスクタイプと一致していることを確認してください。

    FE設定項目`enable_strict_storage_medium_check`が`true`に設定されている場合、テーブルを作成する際にシステムはBEディスクタイプを厳密にチェックします。CREATE TABLEで指定した保管メディアがBEディスクタイプと一致しない場合、「Failed to find enough host in all backends with storage medium is SSD|HDD.」というエラーが返され、テーブルの作成が失敗します。`enable_strict_storage_medium_check`が`false`に設定されている場合、システムはこのエラーを無視し、テーブルを強制的に作成します。ただし、データのロード後にクラスタのディスクスペースが均等に分布されない場合があります。

    v2.3.6、v2.4.2、v2.5.1およびv3.0以降、`storage_medium`が明示的に指定されていない場合、システムはBEディスクタイプに基づいて自動的に保管メディアを推測します。

  - システムは、次のシナリオでこのパラメータをSSDに自動的に設定します:

    - BE（`storage_root_path`）が報告したディスクタイプにSSDのみが含まれている場合
    - BE（`storage_root_path`）が報告したディスクタイプにSSDとHDDの両方が含まれている場合。バージョンv2.3.10、v2.4.5、v2.5.4、およびv3.0以降では、`storage_root_path`がBEにSSDとHDDの両方が含まれ、プロパティ`storage_cooldown_time`が指定されている場合、システムは`storage_medium`をSSDに設定します。

  - システムは、次のシナリオでこのパラメータをHDDに自動的に設定します:

    - BE（`storage_root_path`）が報告したディスクタイプにHDDのみが含まれている場合
    - バージョン2.3.10、2.4.5、2.5.4、および3.0以降では、BEにSSDとHDDの両方が含まれ、プロパティ`storage_cooldown_time`が指定されていない場合、システムは`storage_medium`をHDDに設定します。

- `storage_cooldown_ttl`または`storage_cooldown_time`: 自動保管の冷却時間または時間間隔。自動保管冷却は、SSDからHDDにデータを自動的に移行することを指します。この機能は、初期保管メディアがSSDの場合のみ有効です。

  **パラメータ**

  - `storage_cooldown_ttl`：このテーブルのパーティションの自動保管冷却の**時間間隔**。最新のパーティションをSSDに残したい場合や、古いパーティションを一定の時間経過後にHDDに自動的に冷却したい場合は、このパラメータを使用できます。各パーティションの自動保管冷却時間は、このパラメータの値とパーティションの上限時間の合計値を使用して計算されます。

  サポートされる値は `<num> YEAR`、`<num> MONTH`、`<num> DAY`、`<num> HOUR` です。`<num>`は非負の整数です。デフォルト値はnullで、自動保管冷却が自動的に実行されないことを示します。

  例えば、テーブルを作成する際に値を`"storage_cooldown_ttl"="1 DAY"`と指定し、範囲が`[2023-08-01 00:00:00,2023-08-02 00:00:00)`のパーティション`p20230801`が存在する場合、このパーティションの自動保管冷却時間は`2023-08-02 00:00:00`であり、これは`2023-08-02 00:00:00 + 1 DAY`です。テーブルを作成する際に`"storage_cooldown_ttl"="0 DAY"`と指定した場合、このパーティションの自動保管冷却時間は`2023-08-02 00:00:00`です。

  - `storage_cooldown_time`: テーブルがSSDからHDDに冷却される自動保管の冷却時間（**絶対時間**）。指定した時間は現在時刻より後である必要があります。フォーマット: "yyyy-MM-dd HH:mm:ss"。特定のパーティションに異なるプロパティを構成する必要がある場合は、テーブル作成後に[ALTER TABLE ... ADD PARTITIONまたはALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)を実行してください。

**使用方法**

- 自動保管冷却に関連するパラメータの比較は次のとおりです：
  - `storage_cooldown_ttl`：テーブルのパーティションの自動保管冷却の時間間隔を指定するテーブルプロパティです。システムはパーティションの上限時間に指定された値を加算した時刻で自動保管冷却を実行します。そのため、自動保管冷却はパーティショングラニュリティで実行され、より柔軟です。
- `storage_cooldown_time`: このテーブルの自動ストレージクールダウン時間（**絶対時間**）を指定するテーブルプロパティです。また、テーブル作成後に特定のパーティションに異なるプロパティを設定することもできます。
  - `storage_cooldown_second`: クラスタ内のすべてのテーブルに対する自動ストレージクールダウンレイテンシを指定する静的FEパラメータです。

- テーブルプロパティ `storage_cooldown_ttl` または `storage_cooldown_time` は、FE静的パラメータ `storage_cooldown_second` より優先されます。
- これらのパラメータを構成する際は、「storage_medium = "SSD"」を指定する必要があります。
- これらのパラメータを構成しない場合、自動ストレージクールダウンは自動的に実行されません。
- 各パーティションの自動ストレージクールダウン時間を表示するには、`SHOW PARTITIONS FROM <table_name>` を実行します。

**制限**

- 式とリストパーティショニングはサポートされていません。
- パーティション列は日付型である必要があります。
- 複数のパーティション列はサポートされていません。
- プライマリキーテーブルはサポートされていません。

**パーティションごとの各テーブルのレプリカ数を設定する**

`replication_num`: パーティションごとの各テーブルのレプリカ数。デフォルト数: `3`。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### カラムにBloomフィルターインデックスを追加する

エンジンタイプがOLAPの場合、Bloomフィルターインデックスを採用するカラムを指定できます。

Bloomフィルターインデックスを使用する際の制限事項は以下の通りです:

- データ重複キーやプライマリキーのテーブルのすべての列にBloomフィルターインデックスを作成できます。集約テーブルやユニークキーテーブルの場合、キーカラムにのみBloomフィルターインデックスを作成できます。
- TINYINT、FLOAT、DOUBLE、DECIMALの列はBloomフィルターインデックスの作成をサポートしていません。
- Bloomフィルターインデックスは、`in` や `=` オペレータを含むクエリのパフォーマンスを向上させることができます。このカラムにより、より正確なクエリが行われます。

詳細については、[Bloom filter indexing]を参照してください。(../../../using_starrocks/Bloomfilter_index.md)

```SQL
PROPERTIES (
    "bloom_filter_columns"="k1,k2,k3"
)
```

#### Colocate Joinの使用

Colocate Join属性を使用したい場合は、「properties」に指定します。

```SQL
PROPERTIES (
    "colocate_with"="table1"
)
```

#### ダイナミックパーティションを構成する

ダイナミックパーティション属性を使用したい場合は、プロパティで指定します。

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

| パラメータ                           | 必須     | 説明                                                         |
| ----------------------------------- | -------- | ------------------------------------------------------------ |
| dynamic_partition.enable            | いいえ   | ダイナミックパーティションを有効にするかどうか。有効な値: `TRUE` および `FALSE`。デフォルト値: `TRUE`。 |
| dynamic_partition.time_unit         | はい     | ダイナミックに作成されたパーティションの時間の細かさ。必須パラメータです。有効な値: `DAY`、`WEEK`、`MONTH`。時間の細かさはダイナミックに作成されたパーティションのサフィックス形式を決定します。<br/>- 値が `DAY` の場合、ダイナミック作成されたパーティションのサフィックス形式は `yyyyMMdd` です。例えば、パーティション名のサフィックスは `20200321` です。<br/>- 値が `WEEK` の場合、ダイナミック作成されたパーティションのサフィックス形式は `yyyy_ww` です。例えば、2020年の13週目の場合は `2020_13` です。<br/>- 値が `MONTH` の場合、ダイナミック作成されたパーティションのサフィックス形式は `yyyyMM` です。例えば、`202003` です。 |
| dynamic_partition.start              | いいえ   | ダイナミックパーティションの開始オフセット。このパラメータの値は負の整数である必要があります。このオフセットより前のパーティションは、`dynamic_partition.time_unit` によって決定された現在の日、週、または月に基づいて削除されます。デフォルト値は `Integer.MIN_VALUE` です。つまり、-2147483648 であり、履歴的パーティションは削除されません。 |
| dynamic_partition.end                | はい     | ダイナミックパーティションの終了オフセット。このパラメータの値は正の整数である必要があります。現在の日、週、または月から終了オフセットまでのパーティションが事前に作成されます。 |
| dynamic_partition.prefix             | いいえ   | ダイナミックパーティションの名前に追加されるプレフィックス。デフォルト値: `p`。 |
| dynamic_partition.buckets            | いいえ   | ダイナミックパーティションごとのバケット数。デフォルト値は、予約語 `BUCKETS` によって決定されたバケットの数と同じです。または、StarRocksに自動的に設定されます。

#### ランダムバケット構成されたテーブルのバケットサイズを指定する

v3.2以降、ランダムバケット構成されたテーブルに対して、テーブル作成時に`PROPERTIES`内の`bucket_size`パラメータを使用してバケットサイズを指定できます。デフォルトサイズは `1024 * 1024 * 1024 B`（1 GB） で、最大サイズは 4 GB です。

```sql
PROPERTIES (
    "bucket_size" = "3221225472"
)
```

#### データ圧縮アルゴリズムを設定する

テーブルのデータ圧縮アルゴリズムを指定するには、テーブル作成時に `compression` プロパティを追加します。

`compression` の有効な値は以下の通りです:

- `LZ4`: LZ4アルゴリズム。
- `ZSTD`: Zstandardアルゴリズム。
- `ZLIB`: zlibアルゴリズム。
- `SNAPPY`: Snappyアルゴリズム。

適切なデータ圧縮アルゴリズムの選択方法についての詳細は、[データ圧縮]を参照してください。(../../../table_design/data_compression.md)

#### データローディングのための書き込みクォーラムを設定する

StarRocksクラスタに複数のデータレプリカがある場合、テーブルごとに書き込みクォーラムを設定できます。つまり、StarRocksがローディングタスクが成功したとみなすために必要なレプリカの数を指定できます。`write_quorum` プロパティを追加して、書き込みクォーラムを指定できます。このプロパティはv2.5からサポートされています。

`write_quorum` の有効な値は以下の通りです:

- `MAJORITY`: デフォルト値。データレプリカの**過半数**がローディング成功を返した場合、StarRocksはローディングタスク成功を返します。それ以外の場合、StarRocksはローディングタスク失敗を返します。
- `ONE`: **1つ**のデータレプリカがローディング成功を返した場合、StarRocksはローディングタスク成功を返します。それ以外の場合、StarRocksはローディングタスク失敗を返します。
- `ALL`: **すべて**のデータレプリカがローディング成功を返した場合、StarRocksはローディングタスク成功を返します。それ以外の場合、StarRocksはローディングタスク失敗を返します。

> **注意**
>
> - ローディングに対する低い書き込みクォーラムを設定すると、データのアクセス不能および損失のリスクが増加します。たとえば、2つのレプリカのStarRocksクラスタにテーブルへデータをロードし、データが1つのレプリカにのみ正常にロードされた場合、StarRocksはローディングタスクが成功したと判断しますが、データの存続しているレプリカは1つだけです。ロードされたデータのタブレットを保存しているサーバーがダウンした場合、これらのタブレットのデータはアクセス不能になります。そして、サーバーのディスクが損傷した場合、データが失われます。
> - StarRocksは、すべてのデータレプリカがステータスを返した後に、ローディングタスクのステータスを返します。StarRocksは、ステータスが不明なレプリカがある場合にローディングタスクのステータスを返しません。レプリカ内でのローディングタイムアウトもローディング失敗と見なされます。

#### レプリカ間のデータ書き込みおよびレプリケーションモードを指定する

StarRocksクラスタに複数のデータレプリカがある場合、`PROPERTIES`内の`replicated_storage`パラメータを指定して、レプリカ間のデータ書き込みとレプリケーションモードを構成できます。

- `true` (v3.0以降のデフォルト) は「単一リーダーレプリケーション」を示し、データはプライマリレプリカにのみ書き込まれます。その他のレプリカは、プライマリレプリカからデータを同期します。このモードはデータを複数のレプリカに書き込むことによるCPUコストを大幅に削減します。v2.5からサポートされています。
- `false` (v2.5のデフォルト) は「リーダーレスレプリケーション」を示し、プライマリとセカンダリのレプリカを区別せずに、データを複数のレプリカに直接書き込みます。CPUコストはレプリカの数だけ増加します。

ほとんどの場合、デフォルト値を使用するとデータの書き込み性能が向上します。レプリカ間のデータ書き込みおよびレプリケーションモードを変更したい場合は、`ALTER TABLE` コマンドを実行します。例:

```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
```

#### 一括でロールアップを作成する

テーブルを作成する際に、一括でロールアップを作成できます。

構文:

```SQL
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...)
```

#### View Delta Joinクエリリライトのためのユニークキー制約と外部キー制約を定義する

View Delta Joinシナリオでのクエリリライトを有効にするには、デルタJoinされるテーブルに対してユニークキー制約 `unique_constraints` および外部キー制約 `foreign_key_constraints` を定義する必要があります。詳細については、[非同期マテリアライズドビュー - View Delta Joinシナリオでクエリリライトをリライト]を参照してください。(../../../using_starrocks/query_rewrite_with_materialized_views.md#query-delta-join-rewrite)

```SQL
PROPERTIES (
    "unique_constraints" = "<unique_key>[, ...]",
    "foreign_key_constraints" = "
    (<child_column>[、...])
    REFERENCES 
    [catalog_name].[database_name].<parent_table_name>(<parent_column>[、...])
    [;...]

- `child_column`: テーブルの外部キー。複数の `child_column` を定義できます。
- `catalog_name`: 結合するテーブルが存在するカタログの名前です。このパラメータが指定されていない場合はデフォルトのカタログが使用されます。
- `database_name`: 結合するテーブルが存在するデータベースの名前です。このパラメータが指定されていない場合は現在のデータベースが使用されます。
- `parent_table_name`: 結合するテーブルの名前です。
- `parent_column`: 結合される列です。対応するテーブルの主キーまたは一意なキーである必要があります。

> **注意**
>
> - `unique_constraints` および `foreign_key_constraints` はクエリの書き換えにのみ使用されます。データのロード時に外部キー制約のチェックは保証されません。テーブルにロードされるデータが制約を満たしていることを確認する必要があります。
> - プライマリ キー テーブルのプライマリ キー、または一意なキー テーブルの一意なキーは、デフォルトでそれに対応する `unique_constraints` となります。手動で設定する必要はありません。
> - テーブルの `foreign_key_constraints` 内の `child_column` は、別のテーブルの `unique_constraints` 内の `unique_key` に参照する必要があります。
> - `child_column` と `parent_column` の数は一致している必要があります。
> - `child_column` と対応する `parent_column` のデータ型は一致している必要があります。

#### StarRocks 共有データクラウド向けのクラウドネイティブ テーブルの作成

[StarRocks 共有データクラウドを使用](../../../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)するには、次のプロパティでクラウドネイティブ テーブルを作成する必要があります。

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

- `datacache.enable`: ローカルディスク キャッシュを有効にするかどうか。デフォルト: `true`。

  - このプロパティが `true` に設定されている場合、ロードするデータは同時にオブジェクトストレージとローカルディスク（クエリを高速化するキャッシュとして）に書き込まれます。
  - このプロパティが `false` に設定されている場合、データはオブジェクトストレージのみにロードされます。

  > **注意**
  >
  > ローカルディスク キャッシュを有効にするには、BE 構成項目 `storage_root_path` でディスクのディレクトリを指定する必要があります。詳細については、[BE Configuration items](../../../administration/Configuration.md#be-configuration-items)を参照してください。

- `datacache.partition_duration`: ホットデータの有効期間。ローカルディスク キャッシュが有効な場合、すべてのデータがキャッシュにロードされます。キャッシュがいっぱいになると、StarRocks はキャッシュから最も最近使用されていないデータを削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocks はそのデータが有効期間内かどうかを確認します。データが有効期間内であれば、StarRocks はデータを再度キャッシュにロードします。データが有効期間内でない場合、StarRocks はデータをキャッシュにロードしません。このプロパティは、`YEAR`、`MONTH`、`DAY`、`HOUR` の単位で指定できる文字列値です。たとえば、`7 DAY` や `12 HOUR` を指定できます。指定されていない場合、すべてのデータがホットデータとしてキャッシュされます。

  > **注意**
  >
  > このプロパティは、`datacache.enable` が `true` に設定されている場合のみ使用できます。

- `enable_async_write_back`: データの非同期書き込みを許可するかどうか。デフォルト: `false`。

  - このプロパティが `true` に設定されている場合、ロードタスクはデータがローカルディスク キャッシュに書き込まれるとすぐに成功を返し、データはオブジェクトストレージに非同期で書き込まれます。これによりロード性能が向上しますが、潜在的なシステム障害下でのデータ信頼性のリスクも伴います。
  - このプロパティが `false` に設定されている場合、ロードタスクはデータがオブジェクトストレージとローカルディスク キャッシュの両方に書き込まれた後にのみ成功を返します。これにより、より高い可用性が確保されますが、ロード性能が低下します。

#### 高速なスキーマ進化の設定

`fast_schema_evolution`: テーブルの高速なスキーマ進化を有効にするかどうか。有効な値は `TRUE`（デフォルト）または `FALSE` です。高速なスキーマ進化を有効にすると、列が追加または削除されたときのスキーマ変更の速度が向上し、リソースの使用量が減少します。現在、このプロパティはテーブル作成時にのみ有効にでき、テーブル作成後に[ALTER TABLE](../../sql-statements/data-definition/ALTER_TABLE.md)を使用して変更することはできません。このパラメータは v3.2.0 からサポートされています。

  > **注意**
  >
  > - StarRocks 共有データ クラウドはこのパラメータをサポートしていません。
  > - StarRocks クラスタ内で高速なスキーマ進化を無効にするなど、クラスタレベルで高速なスキーマ進化を構成する必要がある場合は、FE ダイナミックパラメータ [`fast_schema_evolution`](../../../administration/Configuration.md#fast_schema_evolution) を設定できます。

## 例

### ハッシュ バケットおよび列形式のストレージを使用する集計テーブルを作成

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

### 集計テーブルを作成し、格納媒体と冷却時間を設定

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

### レンジパーティション、ハッシュ バケット、列ベースのストレージを使用し、格納媒体と冷却時間を設定する重複キー テーブルを作成

LESS THAN

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

注:

このステートメントは 3 つのデータパーティションを作成します:

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

### MySQL 外部テーブルの作成

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

### HLLカラムを含むテーブルを作成

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

### BITMAP_UNION集約タイプを含むテーブルを作成

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

### Colocate Joinをサポートする2つのテーブルを作成

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

### ビットマップインデックスを持つテーブルを作成

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

### 動的パーティションテーブルを作成

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

### 複数のパーティションの一括作成および整数型の列をパーティション化するテーブルを作成

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

### Hive外部テーブルを作成

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

### プライマリキーを含むテーブルを作成し、ソートキーを指定する

## Reference

- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [USE](USE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [DROP TABLE](DROP_TABLE.md)