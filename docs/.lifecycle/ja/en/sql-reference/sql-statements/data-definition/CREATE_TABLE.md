---
displayed_sidebar: English
---

# CREATE TABLE

## 説明

StarRocksに新しいテーブルを作成します。

> **注記**
>
> この操作には、宛先データベースに対するCREATE TABLE権限が必要です。

## 構文

```plaintext
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...]
[, index_definition1[, index_definition2, ...]])
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

## パラメーター

### column_definition

構文：

```SQL
col_name col_type [agg_type] [NULL | NOT NULL] [DEFAULT "default_value"] [AUTO_INCREMENT] [AS generation_expr]
```

**col_name**: 列名。

通常、`__op`や`__row`で始まる列名はStarRocksで特別な目的に予約されているため、そのような列を作成することはできません。これらの列名を作成すると未定義の動作が発生する可能性があります。それでもこのような列を作成する必要がある場合は、FEの動的パラメーター[`allow_system_reserved_names`](../../../administration/FE_configuration.md#allow_system_reserved_names)を`TRUE`に設定してください。

**col_type**: 列の型。具体的な列情報、例えば型や範囲:

- TINYINT (1バイト): -2^7 + 1 から 2^7 - 1 の範囲。
- SMALLINT (2バイト): -2^15 + 1 から 2^15 - 1 の範囲。
- INT (4バイト): -2^31 + 1 から 2^31 - 1 の範囲。
- BIGINT (8バイト): -2^63 + 1 から 2^63 - 1 の範囲。
- LARGEINT (16バイト): -2^127 + 1 から 2^127 - 1 の範囲。
- FLOAT (4バイト): 科学的記数法がサポートされています。
- DOUBLE (8バイト): 科学的記数法がサポートされています。
- DECIMAL[(精度, スケール)] (16バイト)

  - デフォルト値: DECIMAL(10, 0)
  - 精度: 1 ～ 38
  - スケール: 0 ～ 精度
  - 整数部分: 精度 - スケール

    科学的記数法はサポートされていません。

- DATE (3バイト): 0000-01-01 から 9999-12-31 の範囲。
- DATETIME (8バイト): 0000-01-01 00:00:00 から 9999-12-31 23:59:59 の範囲。
- CHAR[(長さ)]: 固定長文字列。範囲: 1 ～ 255。デフォルト値: 1。
- VARCHAR[(長さ)]: 可変長文字列。デフォルト値は1です。単位: バイト。StarRocks 2.1より前のバージョンでは、`length`の値の範囲は1～65533です。[プレビュー] StarRocks 2.1以降のバージョンでは、`length`の値の範囲は1～1048576です。
- HLL (1～16385バイト): HLL型では、長さやデフォルト値を指定する必要はありません。長さはデータ集約に応じてシステム内で制御されます。HLL列は[hll_union_agg](../../sql-functions/aggregate-functions/hll_union_agg.md)、[hll_cardinality](../../sql-functions/scalar-functions/hll_cardinality.md)、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)でのみクエリまたは使用できます。
- BITMAP: Bitmap型では、指定された長さやデフォルト値は必要ありません。符号なしBIGINT数のセットを表します。最大要素は2^64 - 1までです。

**agg_type**: 集計タイプ。指定されていない場合、この列はキーカラムです。
指定されている場合、値カラムです。サポートされる集計タイプは以下の通りです:

- SUM, MAX, MIN, REPLACE
- HLL_UNION（HLL型のみ）
- BITMAP_UNION（BITMAPのみ）
- REPLACE_IF_NOT_NULL: インポートされたデータがnull以外の値の場合のみ置き換えられることを意味します。null値の場合、StarRocksは元の値を保持します。

> 注記
>
> - BITMAP_UNION集計タイプのカラムをインポートする場合、元のデータ型はTINYINT、SMALLINT、INT、BIGINTである必要があります。
> - テーブル作成時にREPLACE_IF_NOT_NULLカラムでNOT NULLが指定されている場合、StarRocksはユーザーにエラーレポートを送信せずにデータをNULLに変換します。これにより、ユーザーは選択したカラムをインポートできます。

この集計タイプは、key_descタイプがAGGREGATE KEYの集計テーブルにのみ適用されます。

**NULL | NOT NULL**: 列が`NULL`を許容するかどうか。デフォルトでは、Duplicate Key、Aggregate、Unique Keyテーブルを使用するテーブルのすべてのカラムに`NULL`が指定されます。Primary Keyテーブルを使用するテーブルでは、デフォルトで値カラムには`NULL`が、キーカラムには`NOT NULL`が指定されます。生データに`NULL`値が含まれている場合は、`\N`として表現します。StarRocksはデータロード中に`\N`を`NULL`として扱います。

**DEFAULT "default_value"**: カラムのデフォルト値。StarRocksにデータをロードするとき、カラムにマッピングされたソースフィールドが空の場合、StarRocksは自動的にカラムにデフォルト値を設定します。デフォルト値は以下の方法で指定できます:

- **DEFAULT current_timestamp**: 現在の時刻をデフォルト値として使用します。詳細は[current_timestamp()](../../sql-functions/date-time-functions/current_timestamp.md)を参照してください。
- **DEFAULT `<default_value>`**: カラムのデータ型に合った値をデフォルト値として使用します。例えば、カラムのデータ型がVARCHARの場合、`DEFAULT "beijing"`のようにVARCHAR文字列をデフォルト値として指定できます。デフォルト値はARRAY、BITMAP、JSON、HLL、BOOLEANの型にはできません。
- **DEFAULT (\<expr\>)**: 指定された関数によって返される結果をデフォルト値として使用します。[uuid()](../../sql-functions/utility-functions/uuid.md)と[uuid_numeric()](../../sql-functions/utility-functions/uuid_numeric.md)式のみがサポートされています。

**AUTO_INCREMENT**: `AUTO_INCREMENT`カラムを指定します。`AUTO_INCREMENT`カラムのデータ型はBIGINTでなければなりません。自動インクリメントIDは1から始まり、1ステップで増加します。`AUTO_INCREMENT`カラムの詳細については[AUTO_INCREMENT](../../sql-statements/auto_increment.md)を参照してください。v3.0以降、StarRocksは`AUTO_INCREMENT`カラムをサポートしています。

**AS generation_expr**: 生成されるカラムとその式を指定します。[生成されたカラム](../generated_columns.md)を使用すると、式の結果を事前に計算して保存できるため、同じ複雑な式を含むクエリの速度が大幅に向上します。v3.1以降、StarRocksは生成されたカラムをサポートしています。

### index_definition

テーブル作成時にのみビットマップインデックスを作成できます。パラメーターの説明と使用上の注意については、[ビットマップインデックス](../../../using_starrocks/Bitmap_index.md#create-a-bitmap-index)を参照してください。

```SQL
INDEX index_name (col_name[, col_name, ...]) [USING BITMAP] COMMENT 'xxxxxx'
```

### ENGINEタイプ

デフォルト値: `olap`。このパラメーターが指定されていない場合、デフォルトでOLAPテーブル（StarRocksネイティブテーブル）が作成されます。

選択可能な値: `mysql`、`elasticsearch`、`hive`、`hudi`（2.2以降）、`iceberg`、`jdbc`（2.3以降）。外部データソースのクエリに外部テーブルを作成したい場合は、`CREATE EXTERNAL TABLE`を指定し、`ENGINE`にこれらの値のいずれかを設定します。詳細は[外部テーブル](../../../data_source/External_table.md)を参照してください。

**v3.0以降、Hive、Iceberg、Hudi、JDBCデータソースからのデータをクエリするためにカタログの使用を推奨しています。外部テーブルは非推奨です。詳細は[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)、[JDBCカタログ](../../../data_source/catalog/jdbc_catalog.md)を参照してください。**

**v3.1以降、StarRocksはIcebergカタログでParquet形式のテーブルの作成をサポートし、[INSERT INTO](../data-manipulation/INSERT.md)を使用してこれらのParquet形式のIcebergテーブルにデータを挿入できます。詳細は[Icebergテーブルの作成](../../../data_source/catalog/iceberg_catalog.md#create-an-iceberg-table)を参照してください。**

**v3.2以降、StarRocksはHiveカタログでParquet形式のテーブルの作成をサポートし、[INSERT INTO](../data-manipulation/INSERT.md)を使用してこれらのParquet形式のHiveテーブルにデータを挿入できます。詳細は[Hiveテーブルの作成](../../../data_source/catalog/hive_catalog.md#create-a-hive-table)を参照してください。**

- MySQLの場合、以下のプロパティを指定します:

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

    注意：

    MySQLの"table_name"は実際のテーブル名を指す必要があります。対照的に、CREATE TABLEステートメントの"table_name"はStarRocks上でのこのMySQLテーブルの名前を指します。それらは異なる場合も同じ場合もあります。

    StarRocksでMySQLテーブルを作成する目的は、MySQLデータベースにアクセスすることです。StarRocks自体はMySQLデータを保持または保存しません。

- Elasticsearchの場合、以下のプロパティを指定します：

    ```plaintext
    PROPERTIES (

    "hosts" = "http://192.168.0.1:8200,http://192.168.0.2:8200",
    "user" = "root",
    "password" = "root",
    "index" = "tindex",
    "type" = "doc"
    )
    ```

  - `hosts`: Elasticsearchクラスタに接続するために使用されるURLです。1つ以上のURLを指定できます。
  - `user`: 基本認証が有効なElasticsearchクラスタにログインするために使用されるrootユーザーのアカウントです。
  - `password`: 上記rootアカウントのパスワードです。
  - `index`: Elasticsearchクラスタ内のStarRocksテーブルのインデックスです。インデックス名はStarRocksテーブル名と同じです。このパラメータはStarRocksテーブルのエイリアスに設定することができます。
  - `type`: インデックスのタイプです。デフォルト値は`doc`です。

- Hiveの場合、以下のプロパティを指定します：

    ```plaintext
    PROPERTIES (

        "database" = "hive_db_name",
        "table" = "hive_table_name",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
    )
    ```

    ここで、`database`はHiveテーブル内の対応するデータベースの名前です。`table`はHiveテーブルの名前です。`hive.metastore.uris`はサーバーアドレスです。

- JDBCの場合、以下のプロパティを指定します：

    ```plaintext
    PROPERTIES (
    "resource"="jdbc0",
    "table"="dest_tbl"
    )
    ```

    `resource`はJDBCリソース名、`table`は宛先テーブルです。

- Icebergの場合、以下のプロパティを指定します：

   ```plaintext
    PROPERTIES (
    "resource" = "iceberg0", 
    "database" = "iceberg", 
    "table" = "iceberg_table"
    )
    ```

    `resource`はIcebergリソース名、`database`はIcebergデータベース、`table`はIcebergテーブルです。

- Hudiの場合、以下のプロパティを指定します：

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

データは指定されたキーカラムで順序付けされ、キータイプによって異なる属性を持ちます：

- AGGREGATE KEY: キーカラムの同一内容は、指定された集約タイプに従って値カラムに集約されます。これは通常、財務報告や多次元分析などのビジネスシナリオに適用されます。
- UNIQUE KEY/PRIMARY KEY: キーカラムの同一内容は、インポート順に従って値カラムで置き換えられます。キーカラムの追加、削除、変更、およびクエリを行うために適用できます。
- DUPLICATE KEY: キーカラムの同一内容がStarRocksにも同時に存在します。詳細データや集約属性のないデータを格納するために使用できます。**DUPLICATE KEYはデフォルトのタイプです。データはキーカラムに従って順序付けられます。**

> **注意**
>
> AGGREGATE KEY以外のkey_typeを使用してテーブルを作成する場合、値カラムに集約タイプを指定する必要はありません。

### COMMENT

テーブルを作成する際に、オプションとしてテーブルコメントを追加することができます。ただし、COMMENTは`key_desc`の後に配置する必要があります。そうでないと、テーブルを作成できません。

バージョン3.1以降では、`ALTER TABLE <table_name> COMMENT = "new table comment"`を使用してテーブルコメントを変更できます。

### partition_desc

パーティション記述は以下の方法で使用できます：

#### パーティションを動的に作成する

[動的パーティション](../../../table_design/dynamic_partitioning.md)は、パーティションの有効期限(TTL)管理を提供します。StarRocksは自動的に新しいパーティションを事前に作成し、期限切れのパーティションを削除して、データの新鮮さを保証します。この機能を有効にするには、テーブル作成時に動的パーティション関連のプロパティを設定します。

#### パーティションを1つずつ作成する

**パーティションの上限のみを指定する**

構文：

```sql
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
  PARTITION <partition1_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  [ ,
  PARTITION <partition2_name> VALUES LESS THAN ("<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] )
  , ... ] 
)
```

注意：

パーティションには指定されたキーカラムと指定された値の範囲を使用してください。

- パーティション名は[A-z0-9_]のみをサポートします。
- 範囲パーティションのカラムは、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DATE、DATETIMEの型のみをサポートします。
- パーティションは左閉右開です。最初のパーティションの左境界は最小値です。
- NULL値は最小値を含むパーティションにのみ格納されます。最小値を含むパーティションが削除されると、NULL値はもはやインポートできません。
- パーティションカラムは単一カラムまたは複数カラムのいずれかです。パーティション値はデフォルトの最小値です。
- パーティショニングカラムとして1つのカラムのみを指定する場合、最新のパーティションのパーティショニングカラムの上限として`MAXVALUE`を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p1 VALUES LESS THAN ("20210102"),
    PARTITION p2 VALUES LESS THAN ("20210103"),
    PARTITION p3 VALUES LESS THAN MAXVALUE
  )
  ```

ご注意ください：

- パーティションは、時間に関連するデータの管理によく使用されます。
- データのバックトラッキングが必要な場合、後で必要に応じてパーティションを追加するために、最初のパーティションを空にすることを検討してください。

**パーティションの下限と上限の両方を指定する**

構文：

```SQL
PARTITION BY RANGE ( <partitioning_column1> [, <partitioning_column2>, ... ] )
(
    PARTITION <partition_name1> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    [,
    PARTITION <partition_name2> VALUES [( "<lower_bound_for_partitioning_column1>" [ , "<lower_bound_for_partitioning_column2>", ... ] ), ( "<upper_bound_for_partitioning_column1>" [ , "<upper_bound_for_partitioning_column2>", ... ] ) ) 
    , ...]
)
```

注意：

- 固定範囲はLESS THANよりも柔軟性があります。左右のパーティションをカスタマイズできます。
- 固定範囲は、他の点ではLESS THANと同じです。
- パーティショニングカラムとして1つのカラムのみを指定する場合、最新のパーティションのパーティショニングカラムの上限として`MAXVALUE`を設定できます。

  ```SQL
  PARTITION BY RANGE (pay_dt) (
    PARTITION p202101 VALUES [("20210101"), ("20210201")),
    PARTITION p202102 VALUES [("20210201"), ("20210301")),
    PARTITION p202103 VALUES [("20210301"), (MAXVALUE))
  )
  ```

#### バッチで複数のパーティションを作成する

構文：

- パーティション分割カラムが日付型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_date>") END ("<end_date>") EVERY (INTERVAL <N> <time_unit>)
    )
    ```

- パーティション分割カラムが整数型の場合。

    ```sql
    PARTITION BY RANGE (<partitioning_column>) (
        START ("<start_integer>") END ("<end_integer>") EVERY (<partitioning_granularity>)
    )
    ```

説明：

`START()`と`END()`で開始値と終了値を指定し、`EVERY()`で時間単位またはパーティション分割の粒度を指定して、バッチで複数のパーティションを作成できます。

- パーティション分割カラムは日付型または整数型です。
- パーティション分割カラムが日付型の場合、`INTERVAL`キーワードを使用して時間間隔を指定する必要があります。時間単位は、時間（v3.0以降）、日、週、月、年として指定できます。パーティションの命名規則は動的パーティションのそれと同じです。

詳細については、[データ分散](../../../table_design/Data_distribution.md)を参照してください。

### distribution_desc

StarRocksはハッシュバケットとランダムバケットをサポートしています。バケットを設定しない場合、StarRocksはランダムバケットを使用し、デフォルトでバケット数を自動的に設定します。
- ランダムバケッティング（v3.1以降）

  パーティション内のデータについて、StarRocksは特定のカラム値に基づかずに、全てのバケットにランダムにデータを分散します。バケット数をStarRocksに自動的に決定させたい場合は、バケッティングの設定を指定する必要はありません。バケット数を手動で指定する場合、構文は以下の通りです：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <num>
  ```
  
  ただし、ランダムバケッティングによるクエリパフォーマンスは、大量のデータをクエリし、特定のカラムを条件カラムとして頻繁に使用する場合には理想的ではないかもしれません。このシナリオでは、ハッシュバケッティングの使用を推奨します。なぜなら、スキャンおよび計算が必要なバケット数が少なくなるため、クエリパフォーマンスが大幅に向上するからです。

  **注意事項**
  - ランダムバケッティングはDuplicate Keyテーブルの作成にのみ使用できます。
  - ランダムにバケッティングされたテーブルには[Colocation Group](../../../using_starrocks/Colocate_join.md)を指定できません。
  - [Spark Load](../../../loading/SparkLoad.md)はランダムにバケッティングされたテーブルへのデータロードには使用できません。
  - StarRocks v2.5.7以降、テーブルを作成する際にバケット数を設定する必要はありません。StarRocksは自動的にバケット数を設定します。このパラメータを設定したい場合は、[バケット数の設定](../../../table_design/Data_distribution.md#set-the-number-of-buckets)を参照してください。

  詳細情報は[ランダムバケッティング](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

- ハッシュバケッティング

  構文：

  ```SQL
  DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]
  ```

  パーティション内のデータは、バケッティングカラムのハッシュ値とバケット数に基づいてバケットに分割されます。以下の二つの要件を満たすカラムをバケッティングカラムとして選択することを推奨します。

  - IDのような高カーディナリティカラム
  - クエリで頻繁にフィルタとして使用されるカラム

  そのようなカラムが存在しない場合、クエリの複雑さに応じてバケッティングカラムを決定できます。

  - クエリが複雑な場合、バケット間のデータ分散を均等にし、クラスタリソースの利用を向上させるために、高カーディナリティカラムをバケッティングカラムとして選択することを推奨します。
  - クエリが比較的単純な場合、クエリ効率を向上させるために、クエリ条件として頻繁に使用されるカラムをバケッティングカラムとして選択することを推奨します。

  一つのバケッティングカラムを使用してもパーティションデータを各バケットに均等に分散できない場合、複数のバケッティングカラム（最大3つ）を選択できます。詳細は[バケッティングカラムの選択](../../../table_design/Data_distribution.md#hash-bucketing)を参照してください。

  **注意事項**：

  - **テーブルを作成する際には、バケッティングカラムを指定する必要があります**。
  - バケッティングカラムの値は更新できません。
  - バケッティングカラムは指定後に変更することはできません。
  - StarRocks v2.5.7以降、テーブルを作成する際にバケット数を設定する必要はありません。StarRocksは自動的にバケット数を設定します。このパラメータを設定したい場合は、[バケット数の設定](../../../table_design/Data_distribution.md#set-the-number-of-buckets)を参照してください。

### ORDER BY

バージョン3.0以降、プライマリーキーテーブルではプライマリーキーとソートキーが分離されています。ソートキーは`ORDER BY`キーワードによって指定され、任意のカラムの順列と組み合わせが可能です。

> **注意**
>
> ソートキーが指定されている場合、プレフィックスインデックスはソートキーに基づいて構築されます。ソートキーが指定されていない場合、プレフィックスインデックスはプライマリーキーに基づいて構築されます。

### PROPERTIES

#### 初期ストレージ媒体、自動ストレージ冷却時間、レプリカ数の指定

エンジンタイプが`OLAP`の場合、テーブル作成時に初期ストレージ媒体（`storage_medium`）、自動ストレージ冷却時間（`storage_cooldown_time`）または時間間隔（`storage_cooldown_ttl`）、およびレプリカ数（`replication_num`）を指定できます。

プロパティが有効な範囲：テーブルにパーティションが1つのみの場合、プロパティはテーブルに属します。テーブルが複数のパーティションに分割されている場合、プロパティは各パーティションに属します。特定のパーティションに異なるプロパティを設定する必要がある場合は、テーブル作成後に[ALTER TABLE ... ADD PARTITIONまたはALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)を実行できます。

**初期ストレージ媒体と自動ストレージ冷却時間の設定**

```sql
PROPERTIES (
    "storage_medium" = "[SSD|HDD]",
    { "storage_cooldown_ttl" = "<num> { YEAR | MONTH | DAY | HOUR } "
    | "storage_cooldown_time" = "yyyy-MM-dd HH:mm:ss" }
)
```

- `storage_medium`: 初期ストレージ媒体で、`SSD`または`HDD`に設定可能です。明示的に指定したストレージ媒体のタイプが、StarRocksクラスターのBEディスクタイプと一致していることを確認してください（BEの静的パラメータ`storage_root_path`で指定されています）。<br />

    FEの設定項目`enable_strict_storage_medium_check`が`true`に設定されている場合、システムはテーブル作成時にBEディスクタイプを厳密にチェックします。CREATE TABLEで指定したストレージ媒体がBEディスクタイプと一致しない場合、エラー「Failed to find enough host in all backends with storage medium is SSD|HDD.」が返され、テーブル作成が失敗します。`enable_strict_storage_medium_check`が`false`に設定されている場合、システムはこのエラーを無視し、強制的にテーブルを作成します。ただし、データロード後にクラスターのディスクスペースが不均等に分散される可能性があります。<br />

    v2.3.6、v2.4.2、v2.5.1、およびv3.0以降、`storage_medium`が明示的に指定されていない場合、システムはBEディスクタイプに基づいてストレージ媒体を自動的に推測します。<br />

  - 次のシナリオでは、このパラメータは自動的にSSDに設定されます：

    - BEが報告するディスクタイプ（`storage_root_path`）がSSDのみを含む場合。
    - BEが報告するディスクタイプ（`storage_root_path`）がSSDとHDDの両方を含む場合。v2.3.10、v2.4.5、v2.5.4、およびv3.0以降、プロパティ`storage_cooldown_time`が指定されている場合、システムは`storage_medium`をSSDに設定します。

  - 次のシナリオでは、このパラメータは自動的にHDDに設定されます：

    - BEが報告するディスクタイプ（`storage_root_path`）がHDDのみを含む場合。
    - v2.3.10、v2.4.5、v2.5.4、およびv3.0以降、BEが報告するディスクタイプ（`storage_root_path`）がSSDとHDDの両方を含み、プロパティ`storage_cooldown_time`が指定されていない場合、システムは`storage_medium`をHDDに設定します。

- `storage_cooldown_ttl`または`storage_cooldown_time`：自動ストレージ冷却時間または時間間隔です。自動ストレージ冷却とは、SSDからHDDへデータを自動的に移行することを指します。この機能は初期ストレージ媒体がSSDの場合にのみ有効です。

  **パラメータ**

  - `storage_cooldown_ttl`：このテーブルのパーティションの自動ストレージ冷却の時間間隔です。SSD上に最新のパーティションを保持し、一定の時間間隔後に古いパーティションをHDDに自動的に冷却する必要がある場合、このパラメータを使用できます。各パーティションの自動ストレージ冷却時間は、このパラメータの値にパーティションの上限時間を加えて計算されます。

  サポートされる値は`<num> YEAR`、`<num> MONTH`、`<num> DAY`、`<num> HOUR`です。`<num>`は非負の整数です。デフォルト値はnullで、自動的にストレージ冷却が実行されないことを示します。
  例えば、テーブル作成時に`"storage_cooldown_ttl"="1 DAY"`という値を指定し、範囲`[2023-08-01 00:00:00,2023-08-02 00:00:00)`のパーティション`p20230801`が存在する場合、このパーティションの自動ストレージクールダウン時刻は`2023-08-03 00:00:00`となります。これは`2023-08-02 00:00:00 + 1 DAY`です。テーブル作成時に`"storage_cooldown_ttl"="0 DAY"`と指定した場合、このパーティションの自動ストレージクールダウン時刻は`2023-08-02 00:00:00`となります。

  - `storage_cooldown_time`: SSDからHDDへのクールダウン時にテーブルで設定される自動ストレージクールダウン時刻(**絶対時刻**)。指定される時刻は現在時刻よりも後でなければなりません。形式: "yyyy-MM-dd HH:mm:ss"。特定のパーティションに異なるプロパティを設定する必要がある場合は、[ALTER TABLE ... ADD PARTITION または ALTER TABLE ... MODIFY PARTITION](../data-definition/ALTER_TABLE.md)を実行します。

**使用例**

- 自動ストレージクールダウンに関連するパラメータの比較は以下の通りです：
  - `storage_cooldown_ttl`: テーブル内のパーティションに対する自動ストレージクールダウンの時間間隔を指定するテーブルプロパティです。システムは`このパラメータの値 + パーティションの上限時刻`の時刻に自動的にパーティションをクールダウンします。したがって、自動ストレージクールダウンはパーティション単位で行われ、より柔軟です。
  - `storage_cooldown_time`: このテーブルの自動ストレージクールダウン時刻(**絶対時刻**)を指定するテーブルプロパティです。また、テーブル作成後に特定のパーティションに異なるプロパティを設定することもできます。
  - `storage_cooldown_second`: クラスタ内の全テーブルに対する自動ストレージクールダウンの遅延を指定する静的FEパラメータです。

- テーブルプロパティ`storage_cooldown_ttl`または`storage_cooldown_time`は、静的FEパラメータ`storage_cooldown_second`より優先されます。
- これらのパラメータを設定する際には、`"storage_medium = "SSD"`を指定する必要があります。
- これらのパラメータが設定されていない場合、自動ストレージクールダウンは実行されません。
- `SHOW PARTITIONS FROM <table_name>`を実行して、各パーティションの自動ストレージクールダウン時刻を確認できます。

**制限事項**

- 式とリストのパーティションはサポートされていません。
- パーティション列は日付型でなければなりません。
- 複数のパーティション列はサポートされていません。
- プライマリキーテーブルはサポートされていません。

**パーティション内の各タブレットのレプリカ数を設定する**

`replication_num`: パーティション内の各テーブルのレプリカ数。デフォルト値: `3`。

```sql
PROPERTIES (
    "replication_num" = "<num>"
)
```

#### 列にブルームフィルターインデックスを追加

エンジンタイプがOLAPの場合、ブルームフィルターインデックスを使用する列を指定できます。

ブルームフィルターインデックスを使用する際の制限事項は以下の通りです：

- ブルームフィルターインデックスは、Duplicate KeyテーブルやPrimary Keyテーブルの全列に対して作成できます。AggregateテーブルやUnique Keyテーブルでは、キー列に対してのみブルームフィルターインデックスを作成できます。
- TINYINT、FLOAT、DOUBLE、DECIMAL型のカラムはブルームフィルターインデックスの作成に対応していません。
- ブルームフィルターインデックスは、`in`や`=`演算子を含むクエリのパフォーマンスを向上させることができます。例：`Select xxx from table where x in {}`や`Select xxx from table where column = xxx`。この列の値が多様であればあるほど、クエリの精度は向上します。

詳細は[Bloom Filter Indexing](../../../using_starrocks/Bloomfilter_index.md)を参照してください。

```SQL
PROPERTIES (
    "bloom_filter_columns"="k1,k2,k3"
)
```

#### Colocate Joinの使用

Colocate Join属性を使用する場合は、`properties`で指定してください。

```SQL
PROPERTIES (
    "colocate_with"="table1"
)
```

#### 動的パーティションの設定

動的パーティション属性を使用する場合は、`properties`で指定してください。

```SQL
PROPERTIES (

    "dynamic_partition.enable" = "true|false",
    "dynamic_partition.time_unit" = "DAY|WEEK|MONTH",
    "dynamic_partition.start" = "${integer_value}",
    "dynamic_partition.end" = "${integer_value}",
    "dynamic_partition.prefix" = "${string_value}",
    "dynamic_partition.buckets" = "${integer_value}"
)
```

**`PROPERTIES`**

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| dynamic_partition.enable    | いいえ       | 動的パーティションを有効にするかどうか。有効な値: `TRUE` と `FALSE`。デフォルト値: `TRUE`。 |
| dynamic_partition.time_unit | はい      | 動的に作成されるパーティションの時間単位。必須パラメーターです。有効な値: `DAY`, `WEEK`, `MONTH`。時間単位によって動的に作成されるパーティションのサフィックス形式が決まります。<br/> - `DAY`の場合、サフィックス形式は`yyyyMMdd`です。例: `20200321`。<br/> - `WEEK`の場合、サフィックス形式は`yyyy_ww`です。例: `2020_13`は2020年の第13週。<br/> - `MONTH`の場合、サフィックス形式は`yyyyMM`です。例: `202003`。 |
| dynamic_partition.start     | いいえ       | 動的パーティションの開始オフセット。このパラメーターの値は負の整数でなければなりません。このオフセット以前のパーティションは、`dynamic_partition.time_unit`に基づいて現在の日、週、月から削除されます。デフォルト値は`Integer.MIN_VALUE`、つまり-2147483648で、これは過去のパーティションが削除されないことを意味します。 |
| dynamic_partition.end       | はい      | 動的パーティションの終了オフセット。このパラメーターの値は正の整数でなければなりません。現在の日、週、月から終了オフセットまでのパーティションが事前に作成されます。 |
| dynamic_partition.prefix    | いいえ       | 動的パーティションの名前に追加される接頭辞。デフォルト値は`p`です。 |
| dynamic_partition.buckets   | いいえ       | 動的パーティションごとのバケット数。デフォルト値は、予約語`BUCKETS`によって決定されるバケット数またはStarRocksによって自動的に設定されるバケット数と同じです。 |

#### ランダムバケッティングで構成されたテーブルのバケットサイズを指定

v3.2以降、ランダムバケッティングで構成されたテーブルについては、テーブル作成時に`PROPERTIES`内の`bucket_size`パラメーターを使用してバケットサイズを指定し、バケット数のオンデマンドおよび動的な増加を可能にすることができます。単位はBです。

```sql
PROPERTIES (
    "bucket_size" = "1073741824"
)
```

#### データ圧縮アルゴリズムを設定する

テーブル作成時に`compression`プロパティを追加することで、テーブルのデータ圧縮アルゴリズムを指定できます。

`compression`の有効な値は以下の通りです：

- `LZ4`: LZ4アルゴリズム。
- `ZSTD`: Zstandardアルゴリズム。
- `ZLIB`: zlibアルゴリズム。
- `SNAPPY`: Snappyアルゴリズム。

適切なデータ圧縮アルゴリズムの選択方法については、[データ圧縮](../../../table_design/data_compression.md)を参照してください。

#### データロード時の書き込みクォーラムを設定する

StarRocksクラスタに複数のデータレプリカがある場合、テーブルごとに異なる書き込みクォーラムを設定できます。つまり、StarRocksがロードタスクが成功したと判断する前にロード成功を返す必要があるレプリカの数です。テーブル作成時に`write_quorum`プロパティを追加することで書き込みクォーラムを指定できます。このプロパティはv2.5からサポートされています。

`write_quorum`の有効な値は以下の通りです：

- `MAJORITY`：デフォルト値。データレプリカの**過半数**が読み込み成功を返した場合、StarRocksは読み込みタスク成功と判断します。それ以外の場合は、読み込みタスク失敗と判断します。
- `ONE`：データレプリカの**1つ**が読み込み成功を返した場合、StarRocksは読み込みタスク成功と判断します。それ以外の場合は、読み込みタスク失敗と判断します。
- `ALL`：**すべての**データレプリカが読み込み成功を返した場合、StarRocksは読み込みタスク成功と判断します。それ以外の場合は、読み込みタスク失敗と判断します。

> **注意**
>
> - 書き込みクォーラムを低く設定すると、データのアクセス不能やデータ損失のリスクが高まります。例えば、2つのレプリカを持つStarRocksクラスターで、書き込みクォーラムが1のテーブルにデータをロードし、データが1つのレプリカにのみ正常にロードされた場合、StarRocksは読み込みタスクが成功したと判断しますが、データのレプリカは1つしかありません。ロードされたデータのタブレットを格納しているサーバーがダウンした場合、これらのタブレットのデータはアクセスできなくなります。サーバーのディスクが損傷した場合、データは失われます。
> - StarRocksは、すべてのデータレプリカからステータスが返された後にのみ、ロードタスクのステータスを返します。ロード状態が不明なレプリカがある場合、StarRocksはロードタスクのステータスを返しません。レプリカでの読み込みタイムアウトも、読み込み失敗と見なされます。

#### レプリカ間のデータ書き込みとレプリケーションモードを指定する

StarRocksクラスタに複数のデータレプリカがある場合、`PROPERTIES`内の`replicated_storage`パラメータを設定して、レプリカ間のデータ書き込みとレプリケーションモードを指定できます。

- `true`（v3.0以降のデフォルト値）は「シングルリーダーレプリケーション」を意味し、データはプライマリレプリカのみに書き込まれます。他のレプリカはプライマリレプリカからデータを同期します。このモードは、複数のレプリカへのデータ書き込みによるCPUコストを大幅に削減します。v2.5からサポートされています。
- `false`（v2.5のデフォルト値）は「リーダーレスレプリケーション」を意味し、プライマリレプリカとセカンダリレプリカを区別せずに、データが複数のレプリカに直接書き込まれます。CPUコストはレプリカの数に応じて増加します。

ほとんどの場合、デフォルト値を使用することでデータ書き込みのパフォーマンスが向上します。レプリカ間のデータ書き込みとレプリケーションモードを変更したい場合は、ALTER TABLEコマンドを実行します。例えば：

```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
```

#### 一括でロールアップを作成する

テーブルを作成する際に、一括でロールアップを作成できます。

構文：

```SQL
ROLLUP (rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...)
```

#### View Delta Joinクエリの書き換えに向けたUnique Key制約とForeign Key制約の定義

View Delta Joinシナリオでクエリの書き換えを有効にするためには、Delta Joinで結合されるテーブルにUnique Key制約`unique_constraints`とForeign Key制約`foreign_key_constraints`を定義する必要があります。詳細は[非同期マテリアライズドビュー - View Delta Joinシナリオでのクエリ書き換え](../../../using_starrocks/query_rewrite_with_materialized_views.md#query-delta-join-rewrite)を参照してください。

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

- `child_column`：テーブルの外部キーです。複数の`child_column`を定義できます。
- `catalog_name`：結合するテーブルが存在するカタログの名前です。このパラメータが指定されていない場合は、デフォルトカタログが使用されます。
- `database_name`：結合するテーブルが存在するデータベースの名前です。このパラメータが指定されていない場合は、現在のデータベースが使用されます。
- `parent_table_name`：結合するテーブルの名前です。
- `parent_column`：結合される列です。これらは、対応するテーブルのプライマリキーまたはユニークキーでなければなりません。

> **注意**
>
> - `unique_constraints`と`foreign_key_constraints`はクエリの書き換えのためだけに使用されます。データがテーブルにロードされる際の外部キー制約のチェックは保証されません。ロードされるデータが制約を満たしていることを自身で確認する必要があります。
> - プライマリキーテーブルのプライマリキーやユニークキーテーブルのユニークキーは、デフォルトで対応する`unique_constraints`となります。手動で設定する必要はありません。
> - テーブルの`foreign_key_constraints`内の`child_column`は、他のテーブルの`unique_constraints`内の`unique_key`を参照しなければなりません。
> - `child_column`と`parent_column`の数は一致している必要があります。
> - `child_column`と対応する`parent_column`のデータ型は一致している必要があります。

#### StarRocks共有データクラスター用のクラウドネイティブテーブルを作成する

[StarRocks共有データクラスターを使用する](../../../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)ためには、以下のプロパティを持つクラウドネイティブテーブルを作成する必要があります。

```SQL
PROPERTIES (
    "datacache.enable" = "{ true | false }",
    "datacache.partition_duration" = "<string_value>",
    "enable_async_write_back" = "{ true | false }"
)
```

- `datacache.enable`：ローカルディスクキャッシュを有効にするかどうか。デフォルトは`true`です。

  - このプロパティを`true`に設定すると、ロードされるデータはオブジェクトストレージとローカルディスク（クエリ加速のためのキャッシュ）に同時に書き込まれます。
  - このプロパティを`false`に設定すると、データはオブジェクトストレージにのみロードされます。

  > **注記**
  >
  > ローカルディスクキャッシュを有効にするには、BE構成項目`storage_root_path`でディスクのディレクトリを指定する必要があります。詳細は[BE構成項目](../../../administration/BE_configuration.md#be-configuration-items)を参照してください。

- `datacache.partition_duration`：ホットデータの有効期間です。ローカルディスクキャッシュが有効な場合、すべてのデータはキャッシュにロードされます。キャッシュがいっぱいになった場合、StarRocksは使用頻度の低いデータをキャッシュから削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocksはデータが有効期間内かどうかを確認します。データが期間内であれば、StarRocksはキャッシュに再度データをロードします。データが期間外であれば、StarRocksはキャッシュにデータをロードしません。このプロパティは文字列値で、`YEAR`、`MONTH`、`DAY`、`HOUR`などの単位で指定できます。例えば`7 DAY`や`12 HOUR`です。指定されていない場合、すべてのデータはホットデータとしてキャッシュされます。

  > **注記**
  >
  > このプロパティは、`datacache.enable`が`true`に設定されている場合のみ利用可能です。

- `enable_async_write_back`：オブジェクトストレージへの非同期書き戻しを許可するかどうか。デフォルトは`false`です。

  - このプロパティを`true`に設定すると、ロードタスクはデータがローカルディスクキャッシュに書き込まれた時点で成功となり、データはオブジェクトストレージに非同期で書き戻されます。これによりロードパフォーマンスが向上しますが、システム障害が発生した場合にデータの信頼性が損なわれるリスクがあります。
  - このプロパティを`false`に設定すると、ロードタスクはデータがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後に成功となります。これにより可用性が高まりますが、ロードパフォーマンスが低下します。

#### スキーマ進化を高速化する


`fast_schema_evolution`: テーブルのスキーマ進化を高速化するかどうかを指定します。有効な値は `TRUE` または `FALSE`（デフォルト）です。高速スキーマ進化を有効にすると、スキーマ変更の速度が向上し、列の追加や削除時のリソース使用量が削減されます。現在、このプロパティはテーブル作成時にのみ有効化でき、テーブル作成後に [ALTER TABLE](../../sql-statements/data-definition/ALTER_TABLE.md) を使用して変更することはできません。このパラメータは v3.2.0 以降でサポートされています。
  > **注記**
  >
  > - StarRocks の共有データクラスタは、このパラメータをサポートしていません。
  > - StarRocks クラスタ内で高速スキーマ進化を無効にするなど、クラスターレベルで高速スキーマ進化を設定する必要がある場合、FE 動的パラメータ [`enable_fast_schema_evolution`](../../../administration/FE_configuration.md#enable_fast_schema_evolution) を設定できます。

## 例

### ハッシュバケッティングとカラムストレージを使用する集約テーブルを作成する

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

### 集約テーブルを作成し、ストレージ媒体とクールダウン時間を設定する

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
DISTRIBUTED BY HASH(k1, k2)
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
DISTRIBUTED BY HASH(k1, k2)
PROPERTIES(
    "storage_type"="column",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2015-06-04 00:00:00"
);
```

### 範囲パーティション、ハッシュバケッティング、カラムベースストレージを使用する重複キーテーブルを作成し、ストレージ媒体とクールダウン時間を設定する

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

注記：

このステートメントは、以下の3つのデータパーティションを作成します：

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

### HLL 列を含むテーブルを作成する

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

### BITMAP_UNION 集約タイプを含むテーブルを作成する

`v1` および `v2` 列の元のデータ型は TINYINT、SMALLINT、または INT でなければなりません。

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

### Colocate Join をサポートする 2 つのテーブルを作成する

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

### ビットマップインデックスを持つテーブルを作成する

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

### 動的パーティションテーブルを作成する

FE設定で動的パーティション機能を有効にする必要があります（"dynamic_partition.enable" = "true"）。詳細は[動的パーティションの設定](#configure-dynamic-partitions)を参照してください。

この例では、次の3日間のパーティションを作成し、3日前に作成されたパーティションを削除します。例えば、今日が2020-01-08であれば、以下の名前のパーティションが作成されます：p20200108、p20200109、p20200110、p20200111、それぞれの範囲は以下の通りです：

```plaintext
[types: [DATE]; keys: [2020-01-08]; ‥types: [DATE]; keys: [2020-01-09]; )
[types: [DATE]; keys: [2020-01-09]; ‥types: [DATE]; keys: [2020-01-10]; )
[types: [DATE]; keys: [2020-01-10]; ‥types: [DATE]; keys: [2020-01-11]; )
[types: [DATE]; keys: [2020-01-11]; ‥types: [DATE]; keys: [2020-01-12]; )
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

### 一括で複数のパーティションを作成し、パーティションカラムに整数型カラムを指定するテーブルを作成する

次の例では、パーティションカラム `datekey` は INT 型です。すべてのパーティションは、単一のパーティション句 `START ("1") END ("5") EVERY (1)` によって作成されます。すべてのパーティションの範囲は `1` から始まり `5` で終わり、パーティションの粒度は `1` です。
  > **注記**
  >
  > **START()** と **END()** の中のパーティションカラムの値は引用符で囲む必要がありますが、**EVERY()** の中のパーティションの粒度は引用符で囲む必要はありません。

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

### Hive 外部テーブルを作成する

Hive 外部テーブルを作成する前に、Hive リソースとデータベースを作成しておく必要があります。詳細は[外部テーブル](../../../data_source/External_table.md#deprecated-hive-external-table)を参照してください。

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

### 主キーテーブルを作成し、ソートキーを指定する

ユーザーの住所や最終アクティブ時間などの次元からユーザー行動をリアルタイムで分析する必要があるとしましょう。テーブルを作成する際には、`user_id`列をプライマリキーとして定義し、`address`と`last_active`列の組み合わせをソートキーとして定義できます。

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

- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [USE](USE.md)
- [ALTER TABLE](ALTER_TABLE.md)
- [DROP TABLE](DROP_TABLE.md)
