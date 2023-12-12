他のオブジェクトストレージ用のストレージボリュームを作成してデフォルトのストレージボリュームを設定する方法については、[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md) と [SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md) を参照してください。

### データベースとクラウドネイティブテーブルを作成する

デフォルトのストレージボリュームを作成した後は、このストレージボリュームを使用してデータベースとクラウドネイティブテーブルを作成できます。

共有データStarRocksクラスタはすべての[StarRocksテーブルタイプ](../../table_design/table_types/table_types.md)をサポートしています。

次の例では、重複キーのテーブルタイプに基づいてデータベース`cloud_db`およびテーブル`detail_demo`を作成し、ローカルディスクキャッシュを有効にし、ホットデータの有効期間を1か月に設定し、オブジェクトストレージへの非同期データ投入を無効にしています。

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
> 共有データStarRocksクラスタにデータベースまたはクラウドネイティブテーブルを作成する際に、ストレージボリュームが指定されていない場合、デフォルトのストレージボリュームが使用されます。

通常のテーブル`PROPERTIES`に加えて、共有データStarRocksクラスタのテーブルを作成する際には、次の`PROPERTIES`を指定する必要があります。

#### datacache.enable

ローカルディスクキャッシュを有効にするかどうか。

- `true`（デフォルト） このプロパティが`true`に設定されている場合、読み込むデータはオブジェクトストレージとローカルディスク（クエリアクセラレーションのためのキャッシュ）に同時に書き込まれます。
- `false` このプロパティが`false`に設定されている場合、データはオブジェクトストレージにのみロードされます。

> **注意**
>
> バージョン3.0では、このプロパティは`enable_storage_cache`という名前でした。
>
> ローカルディスクキャッシュを有効にするには、CN構成項目`storage_root_path`でディスクのディレクトリを指定する必要があります。

#### datacache.partition_duration

ホットデータの有効期間。ローカルディスクキャッシュが有効な場合、すべてのデータはキャッシュにロードされます。キャッシュがいっぱいになると、StarRocksはキャッシュから最近使用されていないデータを削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocksはデータが有効期間内にあるかどうかをチェックします。データが有効期間内であれば、StarRocksは再びデータをキャッシュにロードします。データが有効期間内でない場合、StarRocksはデータをキャッシュにロードしません。このプロパティは`YEAR`、`MONTH`、`DAY`、`HOUR`の単位で指定できる文字列値です。例: `7 DAY` や `12 HOUR`。指定されていない場合、すべてのデータがホットデータとしてキャッシュされます。

> **注意**
>
> バージョン3.0では、このプロパティは`storage_cache_ttl`という名前でした。
>
> このプロパティは、`datacache.enable`が`true`に設定されている場合にのみ利用可能です。

#### enable_async_write_back

データをオブジェクトストレージに非同期で書き込むかどうか。デフォルト: `false`。
- `true` このプロパティが`true`に設定されている場合、データがローカルディスクキャッシュに書き込まれるとすぐにロードタスクは成功を返し、データがオブジェクトストレージに非同期で書き込まれます。これにより、ロードパフォーマンスが向上しますが、潜在的なシステム障害の下でのデータ信頼性のリスクもあります。
- `false`（デフォルト） このプロパティが`false`に設定されている場合、ロードタスクはデータがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後にのみ成功を返します。これにより、高い可用性が確保されますが、ロードパフォーマンスが低下します。

### テーブル情報を表示する

特定のデータベース内のテーブルの情報は、`SHOW PROC "/dbs/<db_id>"`を使用して表示できます。詳細については、[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)を参照してください。

例:

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共有データStarRocksクラスタのテーブルの`Type`は`CLOUD_NATIVE`です。`StoragePath`フィールドでは、StarRocksはテーブルが保存されているオブジェクトストレージのディレクトリを返します。

### 共有データStarRocksクラスタにデータをロードする

共有データStarRocksクラスタは、StarRocksが提供するすべてのローディング方法をサポートしています。詳細については、[データロードの概要](../../loading/Loading_intro.md)を参照してください。

### 共有データStarRocksクラスタでのクエリ

共有データStarRocksクラスタのテーブルは、StarRocksが提供するすべてのクエリタイプをサポートしています。詳細については、StarRocks [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

> **注意**
>
> 共有データStarRocksクラスタは、[同期マテリアライズドビュー](../../using_starrocks/Materialized_view-single_table.md)をサポートしていません。