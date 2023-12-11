他のオブジェクトストレージのストレージボリュームを作成し、デフォルトのストレージボリュームを設定する方法の詳細については、[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md) および [SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md) を参照してください。

### データベースとクラウドネイティブテーブルの作成

デフォルトのストレージボリュームを作成した後、このストレージボリュームを使用してデータベースおよびクラウドネイティブテーブルを作成できます。

共有データStarRocksクラスタはすべての [StarRocks table types](../../table_design/table_types/table_types.md) をサポートしています。

以下の例では、デフォルトのストレージボリュームを作成して、ローカルディスクキャッシュを有効にし、ホットデータの有効期間を1か月に設定し、オブジェクトストレージへの非同期データ挿入を無効にした後、データベース `cloud_db` とテーブル `detail_demo` を作成します。

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
> 共有データStarRocksクラスタでデータベースまたはクラウドネイティブテーブルを作成する際にストレージボリュームが指定されていない場合、デフォルトのストレージボリュームが使用されます。

通常のテーブル `PROPERTIES` に加えて、共有データStarRocksクラスタのテーブルを作成する際には、以下の `PROPERTIES` を指定する必要があります:

#### datacache.enable

ローカルディスクキャッシュを有効にするかどうか。

- `true` (デフォルト) このプロパティが `true` に設定されている場合、読み込むデータはオブジェクトストレージとローカルディスク（クエリアクセラレーションのためのキャッシュ）に同時に書き込まれます。
- `false` このプロパティが `false` に設定されている場合、データはオブジェクトストレージだけに読み込まれます。

> **注意**
>
> バージョン3.0では、このプロパティは `enable_storage_cache` という名前でした。
>
> ローカルディスクキャッシュを有効にするには、CN構成項目 `storage_root_path` でディスクのディレクトリを指定する必要があります。

#### datacache.partition_duration

ホットデータの有効期間。ローカルディスクキャッシュが有効の場合、すべてのデータがキャッシュに読み込まれます。キャッシュがいっぱいの場合、StarRocksは最近使用されていないデータをキャッシュから削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocksはデータが有効期間内にあるかどうかをチェックします。データが有効期間内であれば、StarRocksは再びデータをキャッシュに読み込みます。データが有効期間内でない場合、StarRocksはデータをキャッシュに読み込みません。このプロパティは、`YEAR`、`MONTH`、`DAY`、`HOUR` などの単位で指定できる文字列値です。たとえば、`7 DAY` や `12 HOUR` です。指定されていない場合、すべてのデータがホットデータとしてキャッシュされます。

> **注意**
>
> バージョン3.0では、このプロパティは `storage_cache_ttl` という名前でした。
>
> このプロパティは、`datacache.enable` が `true` に設定されている場合のみ使用できます。

#### enable_async_write_back

データをオブジェクトストレージに非同期で書き込むかどうか。デフォルト: `false`。
- `true` このプロパティが `true` に設定されている場合、データがローカルディスクキャッシュに書き込まれるとすぐにロードタスクが成功を返し、データはオブジェクトストレージに非同期で書き込まれます。これにより、ロードパフォーマンスが向上しますが、潜在的なシステム障害の下でのデータ信頼性のリスクもあります。
- `false` (デフォルト) このプロパティが `false` に設定されている場合、データはオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後にロードタスクが成功を返します。これにより、高い可用性が保証されますが、ロードパフォーマンスが低下します。

### テーブル情報の表示

`SHOW PROC "/dbs/<db_id>"` を使用して、特定のデータベースのテーブル情報を表示できます。詳細については [SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md) を参照してください。

例:

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共有データStarRocksクラスタのテーブルの `Type` は `CLOUD_NATIVE` です。`StoragePath` のフィールドでは、StarRocks はテーブルが格納されているオブジェクトストレージディレクトリを返します。

### 共有データStarRocksクラスタにデータをロードする

共有データStarRocksクラスタは、StarRocksが提供するすべてのロード方法をサポートしています。詳細については [データロードの概要](../../loading/Loading_intro.md) を参照してください。

### 共有データStarRocksクラスタでのクエリ

共有データStarRocksクラスタのテーブルは、StarRocksが提供するすべての種類のクエリをサポートしています。詳細については StarRocks の [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を参照してください。

> **注意**
>
> 共有データStarRocksクラスタでは、[同期マテリアライズドビュー](../../using_starrocks/Materialized_view-single_table.md) はサポートされていません。