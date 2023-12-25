
他のオブジェクトストレージ用のストレージボリュームを作成し、デフォルトのストレージボリュームを設定する方法については、[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md) および [SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md) を参照してください。

### データベースとクラウドネイティブテーブルの作成

デフォルトのストレージボリュームを作成した後、このストレージボリュームを使用してデータベースとクラウドネイティブテーブルを作成できます。

共有データStarRocksクラスタは、すべての [StarRocksテーブルタイプ](../../table_design/table_types/table_types.md) をサポートしています。

次の例では、Duplicate Keyテーブルタイプに基づいてデータベース `cloud_db` とテーブル `detail_demo` を作成し、ローカルディスクキャッシュを有効にし、ホットデータの有効期間を1ヶ月に設定し、オブジェクトストレージへの非同期データ取り込みを無効にします:

```SQL
CREATE DATABASE cloud_db;
USE cloud_db;
CREATE TABLE IF NOT EXISTS detail_demo (
    recruit_date  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    region_num    TINYINT        COMMENT "範囲 [-128, 127]",
    num_plate     SMALLINT       COMMENT "範囲 [-32768, 32767] ",
    tel           INT            COMMENT "範囲 [-2147483648, 2147483647]",
    id            BIGINT         COMMENT "範囲 [-2^63 + 1 ~ 2^63 - 1]",
    password      LARGEINT       COMMENT "範囲 [-2^127 + 1 ~ 2^127 - 1]",
    name          CHAR(20)       NOT NULL COMMENT "範囲 char(m), mは(1-255) ",
    profile       VARCHAR(500)   NOT NULL COMMENT "上限値 65533バイト",
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

> **注記**
>
> デフォルトのストレージボリュームは、ストレージボリュームが指定されていない場合に、共有データStarRocksクラスタでデータベースまたはクラウドネイティブテーブルを作成する際に使用されます。

共有データStarRocksクラスタでテーブルを作成する際には、通常のテーブル`PROPERTIES`に加えて、以下の`PROPERTIES`を指定する必要があります：

#### datacache.enable

ローカルディスクキャッシュを有効にするかどうか。

- `true`（デフォルト）このプロパティを`true`に設定すると、ロードされるデータはオブジェクトストレージとローカルディスク（クエリ加速のためのキャッシュ）に同時に書き込まれます。
- `false` このプロパティを`false`に設定すると、データはオブジェクトストレージにのみロードされます。

> **注記**
>
> バージョン3.0では、このプロパティは`enable_storage_cache`という名前でした。
>
> ローカルディスクキャッシュを有効にするには、CNの設定項目`storage_root_path`でディスクのディレクトリを指定する必要があります。

#### datacache.partition_duration

ホットデータの有効期間。ローカルディスクキャッシュが有効な場合、すべてのデータはキャッシュにロードされます。キャッシュがいっぱいになったとき、StarRocksは最近使用されていないデータをキャッシュから削除します。クエリが削除されたデータをスキャンする必要がある場合、StarRocksはデータが有効期間内かどうかをチェックします。データが期間内であれば、StarRocksはキャッシュに再度データをロードします。データが期間外であれば、StarRocksはキャッシュにロードしません。このプロパティは文字列で、`YEAR`、`MONTH`、`DAY`、`HOUR`などの単位で指定できます。例えば`7 DAY`や`12 HOUR`です。指定されていない場合、すべてのデータはホットデータとしてキャッシュされます。

> **注記**
>
> バージョン3.0では、このプロパティは`storage_cache_ttl`という名前でした。
>
> このプロパティは`datacache.enable`が`true`に設定されている場合にのみ利用可能です。

#### enable_async_write_back

データをオブジェクトストレージに非同期に書き込むことを許可するかどうか。デフォルト：`false`。
- `true` このプロパティを`true`に設定すると、ロードタスクはデータがローカルディスクキャッシュに書き込まれた時点で成功となり、データはオブジェクトストレージに非同期に書き込まれます。これによりロードパフォーマンスが向上しますが、システム障害が発生した場合にデータの信頼性が損なわれるリスクがあります。
- `false`（デフォルト）このプロパティを`false`に設定すると、ロードタスクはデータがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後にのみ成功となります。これにより可用性が高まりますが、ロードパフォーマンスは低下します。

### テーブル情報の表示

`SHOW PROC "/dbs/<db_id>"`を使用して、特定のデータベース内のテーブル情報を表示できます。詳細は[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)を参照してください。

例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共有データStarRocksクラスタ内のテーブルの`Type`は`CLOUD_NATIVE`です。`StoragePath`フィールドでは、StarRocksはテーブルが保存されているオブジェクトストレージのディレクトリを返します。

### 共有データStarRocksクラスタにデータをロードする

共有データStarRocksクラスタは、StarRocksが提供するすべてのデータロード方法をサポートしています。詳細は[データロードの概要](../../loading/Loading_intro.md)を参照してください。

### 共有データStarRocksクラスタでのクエリ

共有データStarRocksクラスタ内のテーブルは、StarRocksが提供するすべてのクエリタイプをサポートしています。詳細は[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

> **注記**
>
> 共有データStarRocksクラスタは、[同期マテリアライズドビュー](../../using_starrocks/Materialized_view-single_table.md)をサポートしていません。
