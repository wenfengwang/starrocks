---
displayed_sidebar: "Japanese"
---

# 式のパーティショニング（推奨）

v3.0以降、StarRocksは式のパーティショニング（以前は自動パーティショニングとして知られていました）をサポートしており、これはより柔軟で使いやすいものです。このパーティショニング方法は、連続した時間範囲や列挙値に基づいてデータをクエリしたり管理したりする場合など、ほとんどのシナリオに適しています。

テーブルを作成する際にシンプルなパーティション式（時間関数式または列式のいずれか）を指定するだけで済みます。StarRocksはデータのロード中に、データとパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。したがって、テーブル作成時に多数のパーティションを手動で作成する必要はなく、動的パーティションプロパティを構成する必要もありません。

## 時間関数式に基づくパーティショニング

連続した時間範囲に基づいてデータを頻繁にクエリしたり管理したりする場合は、日付型（DATEまたはDATETIME）の列をパーティション列として指定し、時間関数式において年、月、日、または時をパーティションの粒度として指定するだけで済みます。StarRocksはデータの読み込みとパーティション式に基づいてパーティションの開始日時と終了日時または日付を自動的に設定します。

ただし、特定のシナリオでは、例えば過去のデータを月単位のパーティションに、最近のデータを日単位のパーティションに分割したい場合などは、[range partitioning](./Data_distribution_ja.md#range-partitioning)を使用してパーティションを作成する必要があります。

### 構文

```sql
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    { date_trunc ( <time_unit> , <partition_column> ) |
      time_slice ( <partition_column> , INTERVAL <N> <time_unit> [ , boundary ] ) }
```

### パラメータ

| パラメータ            | 必須     | 説明                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `expression`            |     YES     | 現在は、[date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md) および [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 関数のみがサポートされています。`time_slice` 関数を使用する場合は `boundary` パラメータを渡す必要はありません。なぜならばこのシナリオでは、このパラメータのデフォルトで有効な値は `floor` であり、値が `ceil` になることはありません。 |
| `time_unit`             |       YES   | パーティションの粒度を指定します。`hour`、`day`、`month`、または `year` になります。`week` パーティションの粒度はサポートされていません。パーティション粒度が `hour` の場合、パーティション列はDATETIMEデータ型でなければならず、DATEデータ型であってはなりません。 |
| `partition_column` |     YES     | パーティション列の名前。<br/><ul><li>パーティション列はDATEまたはDATETIMEデータ型である必要があります。パーティション列は`NULL`値を許可します。</li><li>`date_trunc`関数を使用する場合、パーティション列はDATEまたはDATETIMEデータ型であっても構いません。`time_slice`関数を使用する場合、パーティション列は必ずDATETIMEデータ型である必要があります。</li><li>パーティション列がDATEデータ型である場合、サポートされる範囲は[0000-01-01 ~ 9999-12-31]となります。パーティション列がDATETIMEデータ型である場合、サポートされる範囲は[0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]となります。</li><li>現時点では1つのパーティション列しか指定できず、複数のパーティション列はサポートされていません。</li></ul> |
| `partition_live_number` |      NO    | 保持される最新のパーティション数です。「最新」とは、パーティションが時系列で並べ替えられ、**現在の日付を基準**としてカウントバックされるパーティション数を指します。その他の古いパーティション（はるか以前に作成されたパーティション）は削除されます。StarRocksはパーティション数の管理をスケジュールし、スケジュール間隔はFEダイナミックパラメータ`dynamic_partition_check_interval_seconds`を通じて構成できます。デフォルトは600秒（10分）です。現在の日付が2023年4月4日であり、「partition_live_number」が「2」に設定されており、パーティションが `p20230401`、 `p20230402`、 `p20230403`、 `p20230404` のような場合、パーティション `p20230403` と `p20230404` が保持され、その他のパーティション（はるか以前に作成されたパーティション）は削除されます。汚れたデータがロードされた場合（たとえば、未来の日付である4月5日や4月6日のデータ）、パーティションには `p20230401`、 `p20230402`、 `p20230403`、 `p20230404`、 `p20230405`、 `p20230406` などが含まれます。その場合、パーティション `p20230403`、 `p20230404`、 `p20230405`、 `p20230406` が保持され、その他のパーティションは削除されます。 |

### 使用上の注記

- データのロード中、StarRocksはデータに基づいていくつかのパーティションを自動的に作成しますが、何らかの理由でロードジョブが失敗した場合、StarRocksによって自動的に作成されたパーティションは自動的に削除されません。
- StarRocksはデフォルトで自動的に作成されるパーティションの最大数を4096としていますが、これは誤って多くのパーティションを作成することを防ぐためのものです。
- パーティションの命名規則は、動的パーティショニングの命名規則と一貫しています。

### **例**

例1: 日単位で頻繁にデータをクエリする場合、パーティション式 `date_trunc()` を使用し、テーブル作成時にパーティション列を `event_day` とし、パーティション粒度を `day` として設定できます。データは自動的に日付に基づいてパーティション分けされます。

```SQL
CREATE TABLE site_access1 (
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY date_trunc('day', event_day)
DISTRIBUTED BY HASH(event_day, site_id);
```

たとえば、次の2つのデータ行がロードされると、StarRocksは自動的に2つのパーティション、`p20230226` と `p20230227`、を作成し、それぞれの範囲を [2023-02-26 00:00:00, 2023-02-27 00:00:00) および [2023-02-27 00:00:00, 2023-02-28 00:00:00) として設定します。その後のデータのロードがこれらの範囲に含まれている場合、それらは自動的に対応するパーティションにルーティングされます。

```SQL
-- 2つのデータ行を挿入
INSERT INTO site_access1  
    VALUES ("2023-02-26 20:12:04",002,"ニューヨーク","サム・スミス",1),
           ("2023-02-27 21:06:54",001,"ロサンゼルス","テイラー・スウィフト",1);

-- パーティションを表示
mysql > SHOW PARTITIONS FROM site_access1;
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range                                                                                                | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 17138       | p20230226     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-26 00:00:00]; ..types: [DATETIME]; keys: [2023-02-27 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
| 17113       | p20230227     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-27 00:00:00]; ..types: [DATETIME]; keys: [2023-02-28 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 行中 (0.00 sec)
```

例2: 最近のパーティションを保持し、過去のパーティションを削除するパーティションライフサイクル管理を実装したい場合は、`partition_live_number` プロパティを使用して保持するパーティション数を指定できます。

```SQL
CREATE TABLE site_access2 (
    event_day DATETIME NOT NULL,
```sql
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY date_trunc('month', event_day)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "partition_live_number" = "3" -- only retains the most recent three partitions
);
```

例3：データをよく週に基づいてクエリする場合、テーブル作成時にパーティション式 `time_slice()` を使用して、パーティション列を `event_day` 、パーティション粒度を7日に設定できます。1週間のデータは1つのパーティションに格納され、パーティションプルーニングを使用してクエリの効率を大幅に向上させることができます。

```SQL
CREATE TABLE site_access(
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY time_slice(event_day, INTERVAL 7 day)
DISTRIBUTED BY HASH(event_day, site_id)
```

## 列式に基づくパーティション設定（v3.1から）

特定のタイプのデータを頻繁にクエリおよび管理する場合、パーティション列を表す列のみを指定する必要があります。StarRocksは、読み込まれたデータのパーティション列の値に基づいて、自動的にパーティションを作成します。

ただし、表に `city` という列が含まれており、国と都市に基づいて頻繁にクエリおよびデータを管理する場合など、同じ国内の複数の都市のデータを1つのパーティションに格納するために [リストパーティション](./list_partitioning.md) を使用する必要があります。

### 構文

```sql
PARTITION BY expression
...
[ PROPERTIES("partition_live_number" = "xxx") ]

expression ::=
    ( partition_columns )
    
partition_columns ::=
    <column>, [ <column> [,...] ]
```

### パラメータ

| **パラメータ**        | **必須** | **説明**                                                  |
| --------------------- | -------- | ------------------------------------------------------- |
| `partition_columns`   | YES      | パーティション列の名前。<br/> <ul><li>パーティション列の値は、文字列（BINARYはサポートされていません）、日付または日時、整数、および真偽値である必要があります。パーティション列は `NULL` 値を許可します。</li><li>各パーティションにはパーティション列の値が同じデータのみを含めることができます。パーティション内に異なる値を持つデータを含めるには、 [リストパーティション](./list_partitioning.md) を参照してください。</li></ul> |
| `partition_live_number` | NO       | 保持するパーティションの数。パーティション間でパーティション列の値を比較し、定期的に小さい値を持つパーティションを削除し、大きい値を持つパーティションを保持します。<br/>StarRocksは、パーティションの数を管理するためのタスクをスケジュールし、スケジュール間隔は FE パラメータの `dynamic_partition_check_interval_seconds` を介して構成できます。デフォルトは600秒（10分）です。<br/>**注意**<br/>パーティション列の値が文字列の場合、StarRocksはパーティション名の辞書順を比較し、定期的に後に来るパーティションを削除し、早く来るパーティションを保持します。 |

### 使用上の注意

- データの読み込み中、StarRocksは読み込まれたデータに基づいていくつかのパーティションを自動的に作成しますが、読み込みジョブが何らかの理由で失敗した場合、StarRocksによって自動的に作成されたパーティションは自動的に削除できません。
- StarRocksは、自動的に作成されるパーティションのデフォルト最大数を4096に設定していますが、これは誤って多くのパーティションを作成することを防ぐためのものです。
- パーティションの命名規則: 複数のパーティション列が指定されている場合、異なるパーティション列の値はパーティション名で下線 `_` で連結され、フォーマットは `p<パーティション列1の値>_<パーティション列2の値>_...` です。 たとえば、`dt` と `province` の2つの列がパーティション列として指定され、その両方が文字列型であり、値が `2022-04-01` および `beijing` のデータ行がロードされる場合、自動的に作成される対応するパーティションは `p20220401_beijing` という名前が付けられます。

### 例

例1：データセンターの課金の詳細を頻繁に日付範囲と特定の都市に基づいてクエリする場合、テーブル作成時にパーティション式を使用して、最初のパーティション列を `dt` と `city` と指定できます。これにより、同じ日付と都市に属するデータが同じパーティションにルーティングされ、パーティションプルーニングを使用してクエリの効率を大幅に向上させることができます。

```SQL
CREATE TABLE t_recharge_detail1 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY (dt,city)
DISTRIBUTED BY HASH(`id`);
```

テーブルに単一のデータ行を挿入します。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

パーティションを表示します。結果は、StarRocksが読み込まれたデータに基づいて自動的にパーティション `p20220401_Houston` を作成したことを示しています。以降の読み込み中、`dt` と `city` のパーティション列に値 `2022-04-01` と `Houston` を持つデータはこのパーティションに格納されます。

> **注意**
>
> 各パーティションには、パーティション列の値に指定された1つの値のデータのみを含めることができます。パーティション内にパーティション列の値に複数の値を指定する場合は、[リストパーティション](./list_partitioning.md) を参照してください。

```SQL
MySQL > SHOW PARTITIONS from t_recharge_detail1\G
*************************** 1. row ***************************
             PartitionId: 16890
           PartitionName: p20220401_Houston
          VisibleVersion: 2
      VisibleVersionTime: 2023-07-19 17:24:53
      VisibleVersionHash: 0
                   State: NORMAL
            PartitionKey: dt, city
                    List: (('2022-04-01', 'Houston'))
         DistributionKey: id
                 Buckets: 6
          ReplicationNum: 3
           StorageMedium: HDD
            CooldownTime: 9999-12-31 23:59:59
LastConsistencyCheckTime: NULL
                DataSize: 2.5KB
              IsInMemory: false
                RowCount: 1
1 row in set (0.00 sec)
```

例2：パーティションのライフサイクル管理のために、テーブル作成時に "`partition_live_number` プロパティを設定することもできます。たとえば、テーブルが最新の3つのパーティションのみを保持するよう指定できます。

```SQL
CREATE TABLE t_recharge_detail2 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY (dt,city)
DISTRIBUTED BY HASH(`id`) 
PROPERTIES(
    "partition_live_number" = "3" -- only retains the most recent three partitions
);
```

## パーティションの管理

### パーティションにデータをロードする

データのロード中、StarRocksはロードされたデータとパーティション式によるパーティションルールに基づいて、自動的にパーティションを作成します。

パーティション式を使用して表を作成し、特定のパーティションでデータを上書きするために [INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select) を使用し、パーティションが作成されているかどうかに関係なく、`PARTITION()` 内で明示的にパーティションの範囲を提供する必要があります。これは [範囲パーティション](./Data_distribution.md#range-partitioning) や [リストパーティション](./list_partitioning.md) とは異なり、`PARTITION (<partition_name>)` でパーティション名だけを提供できます。

テーブル作成時に時間関数式を使用し、特定のパーティションでデータを上書きする場合、そのパーティションの開始日または日時（テーブル作成時に設定されたパーティション粒度）を指定する必要があります。パーティションが存在しない場合、データのロード中に自動的に作成されます。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

表の作成時に列式を使用し、特定のパーティションでデータを上書きする場合、パーティションに含まれるパーティション列の値を提供する必要があります。パーティションが存在しない場合、データのロード中に自動的に作成されます。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### パーティションを表示する

自動的に作成されたパーティションの特定の情報を表示する場合は、`SHOW PARTITIONS FROM <table_name>` 文を使用する必要があります。`SHOW CREATE TABLE <table_name>` 文は、テーブル作成時に構成された節式パーティションの構文のみを返します。
| PartitionId | PartitionName     | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | List                        | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 16890       | p20220401_Houston | 2              | 2023-07-19 17:24:53 | 0                  | NORMAL | dt, city     | (('2022-04-01', 'Houston')) | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
| 17056       | p20220402_texas   | 2              | 2023-07-19 17:27:42 | 0                  | NORMAL | dt, city     | (('2022-04-02', 'texas'))   | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)

## Limits

- StarRocksの[共有データモード](../deployment/shared_data/s3.md)はv3.1.0から[時間関数式](#partitioning-based-on-a-time-function-expression)をサポートしています。さらに、v3.1.1からはStarRocksの[共有データモード](../deployment/shared_data/s3.md)は[列式](#partitioning-based-on-the-column-expression-since-v31)をさらにサポートしています。
- 現在、CTASを使用して設定された式のパーティションを作成することはできません。
- 現在、式のパーティションを使用するテーブルにデータをロードするには、Spark Loadを使用することはできません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>`ステートメントを使用して列式を使用して作成されたパーティションを削除する場合、パーティション内のデータは直接削除され、復元することはできません。
- 現在、式のパーティションで作成したパーティションを[バックアップおよび復元](../administration/Backup_and_restore.md)することはできません。