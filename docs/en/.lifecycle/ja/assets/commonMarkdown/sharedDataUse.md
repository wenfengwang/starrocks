他のオブジェクトストレージのストレージボリュームを作成し、デフォルトのストレージボリュームを設定する方法の詳細については、[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)と[SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)を参照してください。

### データベースとクラウドネイティブテーブルの作成

デフォルトのストレージボリュームを作成した後、このストレージボリュームを使用してデータベースとクラウドネイティブテーブルを作成できます。

共有データのStarRocksクラスタは、すべての[StarRocksテーブルタイプ](../../table_design/table_types/table_types.md)をサポートしています。

次の例では、Duplicate Keyテーブルタイプに基づいてデータベース`cloud_db`とテーブル`detail_demo`を作成し、ローカルディスクキャッシュを有効にし、ホットデータの有効期間を1ヶ月に設定し、非同期データのインジェストをオブジェクトストレージに無効にします。

```SQL
CREATE DATABASE cloud_db;
USE cloud_db;
CREATE TABLE IF NOT EXISTS detail_demo (
    recruit_date  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    region_num    TINYINT        COMMENT "range [-128, 127]",
    num_plate     SMALLINT       COMMENT "range [-32768, 32767] ",
    tel           INT            COMMENT "range [-2147483648, 2147483647]",
    id            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
    password      LARGEINT       COMMENT "range [-2^127 + 1 ~ 2^127 - 1]",
    name          CHAR(20)       NOT NULL COMMENT "range char(m),m in (1-255) ",
    profile       VARCHAR(500)   NOT NULL COMMENT "upper limit value 65533 bytes",
    ispass        BOOLEAN        COMMENT "true/false")
DUPLICATE KEY(recruit_date, region_num)
DISTRIBUTED BY HASH(recruit_date, region_num)
PROPERTIES (
    "storage_volume" = "def_volume",
    "datacache.enable" = "true",
    "datacache.partition_duration" = "1 MONTH",
    "enable_async_write_back" = "false"
);
```

> **注意**
>
> ストレージボリュームが指定されていない場合、共有データのStarRocksクラスタでデータベースまたはクラウドネイティブテーブルを作成する際には、デフォルトのストレージボリュームが使用されます。

共有データのStarRocksクラスタのテーブルを作成する際には、通常のテーブル`PROPERTIES`に加えて、次の`PROPERTIES`を指定する必要があります。

#### datacache.enable

ローカルディスクキャッシュを有効にするかどうか。

- `true`（デフォルト）このプロパティが`true`に設定されている場合、読み込むデータはオブジェクトストレージとローカルディスク（クエリの高速化のためのキャッシュとして）に同時に書き込まれます。
- `false`このプロパティが`false`に設定されている場合、データはオブジェクトストレージにのみ読み込まれます。

> **注意**
>
> バージョン3.0では、このプロパティの名前は`enable_storage_cache`でした。
>
> ローカルディスクキャッシュを有効にするには、CNの設定項目`storage_root_path`でディスクのディレクトリを指定する必要があります。

#### datacache.partition_duration

ホットデータの有効期間。ローカルディスクキャッシュが有効になっている場合、すべてのデータがキャッシュに読み込まれます。キャッシュがいっぱいになると、StarRocksは最近使用されていないデータをキャッシュから削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocksはデータが有効期間内かどうかをチェックします。データが有効期間内であれば、StarRocksはデータを再びキャッシュに読み込みます。データが有効期間内でない場合、StarRocksはデータをキャッシュに読み込みません。このプロパティは、`YEAR`、`MONTH`、`DAY`、`HOUR`の単位で指定できる文字列値です。例えば、`7 DAY`や`12 HOUR`などです。指定されていない場合、すべてのデータがホットデータとしてキャッシュされます。

> **注意**
>
> バージョン3.0では、このプロパティの名前は`storage_cache_ttl`でした。
>
> このプロパティは、`datacache.enable`が`true`に設定されている場合にのみ使用できます。

#### enable_async_write_back

データを非同期でオブジェクトストレージに書き込むことを許可するかどうか。デフォルト：`false`。
- `true`このプロパティが`true`に設定されている場合、データがローカルディスクキャッシュに書き込まれるとすぐにロードタスクは成功を返し、データは非同期でオブジェクトストレージに書き込まれます。これにより、読み込みパフォーマンスが向上しますが、潜在的なシステム障害下でのデータ信頼性のリスクもあります。
- `false`（デフォルト）このプロパティが`false`に設定されている場合、データがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後にのみ、ロードタスクは成功を返します。これにより、高い可用性が保証されますが、読み込みパフォーマンスが低下します。

### テーブル情報の表示

特定のデータベース内のテーブルの情報を`SHOW PROC "/dbs/<db_id>"`を使用して表示することができます。詳細については、[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)を参照してください。

例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共有データのStarRocksクラスタのテーブルの`Type`は`CLOUD_NATIVE`です。`StoragePath`のフィールドでは、StarRocksはテーブルが格納されているオブジェクトストレージのディレクトリを返します。

### 共有データのStarRocksクラスタにデータをロードする

共有データのStarRocksクラスタは、StarRocksが提供するすべてのロード方法をサポートしています。詳細については、[データロードの概要](../../loading/Loading_intro.md)を参照してください。

### 共有データのStarRocksクラスタでのクエリ

共有データのStarRocksクラスタのテーブルは、StarRocksが提供するすべてのクエリタイプをサポートしています。詳細については、StarRocksの[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。
