
他のリモートストレージ用のストレージボリュームを作成し、デフォルトのストレージボリュームを設定する方法については、[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md) および [SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md) を参照してください。

### データベースとクラウドネイティブテーブルの作成

デフォルトのストレージボリュームを作成した後、そのストレージボリュームを使用してデータベースとクラウドネイティブテーブルを作成できます。

StarRocks の分離されたストレージと計算クラスターは、すべての[データモデル](../../table_design/table_types/table_types.md)をサポートしています。

以下の例では、データベース `cloud_db` を作成し、詳細モデルに基づいてテーブル `detail_demo` を作成し、ローカルディスクキャッシュを有効にし、ホットデータの有効期限を1か月に設定し、非同期データインポートを無効にします：

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

> **説明**
>
> StarRocks の分離されたストレージと計算クラスターでデータベースやクラウドネイティブテーブルを作成する際、ストレージボリュームを指定しない場合、StarRocks はデフォルトのストレージボリュームを使用します。

通常のテーブルの PROPERTIES に加えて、テーブルを作成する際に以下の PROPERTIES を指定する必要があります：

#### datacache.enable

ローカルディスクキャッシュを有効にするかどうか。デフォルト値は `true` です。

- `true`：データはオブジェクトストレージ（または HDFS）とローカルディスク（クエリの加速のためのキャッシュとして）に同時にインポートされます。
- `false`：データはオブジェクトストレージにのみインポートされます。

> **説明**
>
> - バージョン v3.0 では、このパラメータの名前は `enable_storage_cache` です。
> - ローカルディスクキャッシュを有効にするには、CN の設定項目 `storage_root_path` でディスクディレクトリを指定する必要があります。

#### datacache.partition_duration

ホットデータの有効期限。ローカルディスクキャッシュが有効な場合、すべてのデータはローカルディスクキャッシュにインポートされます。キャッシュがいっぱいになった場合、StarRocks は最近あまり使用されていない（Less recently used）データをキャッシュから削除します。削除されたデータをスキャンする必要があるクエリがある場合、StarRocks はそのデータが有効期限内にあるかどうかをチェックします。データが有効期限内であれば、StarRocks はデータを再びキャッシュにインポートします。データが有効期限外であれば、StarRocks はキャッシュにインポートしません。この属性は文字列で、`YEAR`、`MONTH`、`DAY`、`HOUR` などの単位で指定できます。例えば `7 DAY` や `12 HOUR` です。指定しない場合、StarRocks はすべてのデータをホットデータとしてキャッシュします。

> **説明**
>
> - バージョン v3.0 では、このパラメータの名前は `storage_cache_ttl` です。
> - `datacache.enable` が `true` に設定されている場合にのみ、この属性が使用できます。

#### enable_async_write_back

データをオブジェクトストレージに非同期で書き込むことを許可するかどうか。デフォルト値は `false` です。

- `true`：インポートタスクは、データがローカルディスクキャッシュに書き込まれた後すぐに成功を返し、データはオブジェクトストレージに非同期で書き込まれます。データの非同期書き込みを許可することでインポート性能を向上させることができますが、システムに障害が発生した場合、データの信頼性に一定のリスクが生じる可能性があります。
- `false`：データがオブジェクトストレージとローカルディスクキャッシュの両方に書き込まれた後にのみ、インポートタスクは成功を返します。データの非同期書き込みを無効にすることで、より高い可用性が保証されますが、インポート性能が低下する可能性があります。

### テーブル情報の表示

`SHOW PROC "/dbs/<db_id>"` を使用して、特定のデータベース内のテーブル情報を表示できます。詳細については、[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md) を参照してください。

例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

StarRocks の分離されたストレージと計算クラスター内のテーブルの `Type` は `CLOUD_NATIVE` です。`StoragePath` フィールドはテーブルがオブジェクトストレージにあるパスです。

### StarRocks の分離されたストレージと計算クラスターへのデータのインポート

StarRocks の分離されたストレージと計算クラスターは、StarRocks が提供するすべてのインポート方法をサポートしています。詳細については、[インポートの概要](../../loading/Loading_intro.md) を参照してください。

### StarRocks の分離されたストレージと計算クラスターでのクエリ

StarRocks の分離されたストレージと計算クラスターは、StarRocks が提供するすべてのクエリ方法をサポートしています。詳細については、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を参照してください。

> **説明**
>
> StarRocks の分離されたストレージと計算クラスターは、現在[同期マテリアライズドビュー](../../using_starrocks/Materialized_view-single_table.md)をサポートしていません。
