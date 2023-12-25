---
displayed_sidebar: English
---

# 式によるパーティショニング（推奨）

v3.0以降、StarRocksは式によるパーティショニング（以前は自動パーティショニングと呼ばれていた）をサポートしており、これはより柔軟でユーザーフレンドリーです。このパーティショニング方法は、連続した時間範囲やenum値に基づいてデータをクエリや管理するなど、ほとんどのシナリオに適しています。

テーブル作成時には、単純なパーティション式（時間関数式またはカラム式）を指定するだけで済みます。データロード時に、StarRocksはデータとパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。テーブル作成時に多数のパーティションを手動で作成したり、動的パーティションプロパティを設定したりする必要はありません。

## 時間関数式に基づくパーティショニング

連続する時間範囲に基づいてデータを頻繁にクエリや管理する場合、日付型（DATEまたはDATETIME）のカラムをパーティションカラムとして指定し、時間関数式でパーティションの粒度として年、月、日、または時を指定するだけで済みます。StarRocksは、ロードされたデータとパーティション式に基づいて自動的にパーティションを作成し、パーティションの開始日と終了日時を設定します。

ただし、履歴データを月単位でパーティションに分割し、最近のデータを日単位でパーティションに分割するなど、特定のシナリオでは、[範囲パーティショニング](./Data_distribution.md#range-partitioning)を使用してパーティションを作成する必要があります。

### 構文

```sql
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    { date_trunc ( <time_unit> , <partition_column> ) |
      time_slice ( <partition_column> , INTERVAL <N> <time_unit> [ , boundary ] ) }
```

### パラメーター

| パラメーター              | 必須 | 説明                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `expression`            |     はい     | 現在、[date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md)関数と[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)関数のみがサポートされています。`time_slice`関数を使用する場合、`boundary`パラメーターを渡す必要はありません。これは、このシナリオでは、このパラメーターのデフォルト値および有効な値が`floor`であり、`ceil`にすることはできないためです。 |
| `time_unit`             |       はい   | パーティションの粒度で、`hour`、`day`、`month`、`year`があります。`week`のパーティション粒度はサポートされていません。パーティション粒度が`hour`の場合、パーティションカラムはDATETIMEデータ型でなければならず、DATEデータ型ではいけません。 |
| `partition_column` |     はい     | パーティションカラムの名前。<br/><ul><li>パーティションカラムはDATEまたはDATETIMEデータ型のみです。`NULL`値が許可されます。</li><li>`date_trunc`関数を使用する場合、パーティションカラムはDATEまたはDATETIMEデータ型にできます。`time_slice`関数を使用する場合、パーティションカラムはDATETIMEデータ型でなければなりません。</li><li>DATEデータ型の場合、サポートされる範囲は[0000-01-01 ~ 9999-12-31]です。DATETIMEデータ型の場合、サポートされる範囲は[0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]です。</li><li>現在、複数のパーティションカラムはサポートされておらず、指定できるパーティションカラムは1つだけです。</li></ul> |
| `partition_live_number` |      いいえ    | 保持される最新のパーティションの数です。「最近」とは、**現在の日付を基準に**時系列順に並べられたパーティションの中で、逆順に数えたパーティションが保持され、それ以前に作成されたパーティションは削除されることを意味します。StarRocksはパーティションの数を管理するタスクをスケジュールし、スケジューリング間隔はFEの動的パラメーター`dynamic_partition_check_interval_seconds`で設定でき、デフォルトは600秒（10分）です。例えば、現在の日付が2023年4月4日で`partition_live_number`が`2`に設定されている場合、パーティションに`p20230401`、`p20230402`、`p20230403`、`p20230404`が含まれているとします。その場合、`p20230403`と`p20230404`のパーティションが保持され、他のパーティションは削除されます。未来の日付である4月5日と4月6日のデータがロードされた場合、パーティションには`p20230401`、`p20230402`、`p20230403`、`p20230404`、`p20230405`、`p20230406`が含まれているとします。その場合、`p20230403`、`p20230404`、`p20230405`、`p20230406`のパーティションが保持され、他のパーティションは削除されます。 |

### 使用上の注意

- データロード中、StarRocksはロードされたデータに基づいて自動的にいくつかのパーティションを作成しますが、何らかの理由でロードジョブが失敗した場合、StarRocksによって自動的に作成されたパーティションは自動的に削除されません。
- StarRocksは自動的に作成されるパーティションの最大数をデフォルトで4096に設定しており、これはFEパラメーター`max_automatic_partition_number`で設定できます。このパラメーターにより、誤って多数のパーティションを作成することを防ぐことができます。
- パーティションの命名規則は動的パーティショニングの命名規則と一致しています。

### **例**

例1: 日ごとにデータを頻繁にクエリする場合、テーブル作成時にパーティション式`date_trunc()`を使用し、パーティションカラムを`event_day`、パーティションの粒度を`day`に設定できます。データはロード中に日付に基づいて自動的にパーティション分割され、同じ日のデータは1つのパーティションに格納されます。これにより、パーティションプルーニングを使用してクエリ効率を大幅に向上させることができます。

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


たとえば、次の 2 つのデータ行がロードされた場合、StarRocks は自動的に `p20230226` と `p20230227` の 2 つのパーティションを作成します。それぞれの範囲は [2023-02-26 00:00:00, 2023-02-27 00:00:00) と [2023-02-27 00:00:00, 2023-02-28 00:00:00) です。その後にロードされるデータがこれらの範囲内にある場合、それらは自動的に対応するパーティションにルーティングされます。

```SQL
-- 2 つのデータ行を挿入
INSERT INTO site_access1  
    VALUES ("2023-02-26 20:12:04",002,"New York","Sam Smith",1),
           ("2023-02-27 21:06:54",001,"Los Angeles","Taylor Swift",1);

-- パーティションを表示
mysql > SHOW PARTITIONS FROM site_access1;
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range                                                                                                | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 17138       | p20230226     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-26 00:00:00]; ..types: [DATETIME]; keys: [2023-02-27 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
| 17113       | p20230227     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-27 00:00:00]; ..types: [DATETIME]; keys: [2023-02-28 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

例 2: パーティションのライフサイクル管理を実装したい場合、つまり最近のパーティションのみを保持して古いパーティションを削除する場合は、`partition_live_number` プロパティを使用して保持するパーティションの数を指定できます。

```SQL
CREATE TABLE site_access2 (
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
) 
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY date_trunc('month', event_day)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "partition_live_number" = "3" -- 最新の 3 つのパーティションのみを保持
);
```

例 3: 週単位でデータを頻繁にクエリする場合、`time_slice()` 関数を使用して、テーブル作成時にパーティション列 `event_day` を 7 日間隔で設定することができます。1 週間分のデータが 1 つのパーティションに格納され、パーティションプルーニングを使用してクエリ効率を大幅に向上させることができます。

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

## 列式に基づくパーティショニング (v3.1 以降)

特定のタイプのデータを頻繁にクエリおよび管理する場合、そのタイプを表す列をパーティション列として指定するだけで済みます。StarRocks は、ロードされたデータのパーティション列の値に基づいて自動的にパーティションを作成します。

ただし、テーブルに `city` 列が含まれており、国や都市に基づいてデータを頻繁にクエリおよび管理する必要がある特殊なシナリオでは、[リストパーティショニング](./list_partitioning.md) を使用して同じ国の複数の都市のデータを 1 つのパーティションに格納する必要があります。

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

### パラメーター

| **パラメーター**          | **必須** | **説明**                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `partition_columns`     | はい      | パーティション列の名前。<br/> <ul><li>パーティション列の値は、文字列（BINARY はサポートされていません）、日付または日時、整数、およびブール値が可能です。パーティション列は `NULL` 値を許容します。</li><li> 各パーティションは、パーティション列で同じ値を持つデータのみを含むことができます。パーティション列で異なる値を持つデータを同一パーティションに含めるには、[リストパーティショニング](./list_partitioning.md)を参照してください。</li></ul> |
| `partition_live_number` | いいえ      | 保持されるパーティションの数。パーティション間でパーティション列の値を比較し、小さい値のパーティションを定期的に削除しながら、大きい値のパーティションを保持します。<br/>StarRocks はパーティション数を管理するタスクをスケジュールし、そのスケジューリング間隔は FE の動的パラメーター `dynamic_partition_check_interval_seconds` で設定でき、デフォルトは 600 秒（10 分）です。<br/>**注記**<br/>パーティション列の値が文字列の場合、StarRocks はパーティション名の辞書順を比較し、定期的に順序が前のパーティションを保持し、後のパーティションを削除します。 |

### 使用上の注意

- データロード中、StarRocks はロードされたデータに基づいて自動的にいくつかのパーティションを作成しますが、何らかの理由でロードジョブが失敗した場合、StarRocks によって自動的に作成されたパーティションは自動的に削除されません。
- StarRocks は自動的に作成されるパーティションのデフォルト最大数を 4096 に設定しています。これは FE のパラメーター `max_automatic_partition_number` で設定可能です。このパラメーターにより、誤って多数のパーティションを作成することを防ぐことができます。

- パーティションの命名規則: 複数のパーティション列が指定されている場合、異なるパーティション列の値はアンダースコア `_` で接続され、形式は `p<partition column 1 の値>_<partition column 2 の値>_...` となります。例えば、`dt` と `province` の2つの列がパーティション列として指定されていて（どちらも文字列型）、値 `2022-04-01` と `beijing` のデータ行がロードされた場合、自動的に作成される対応するパーティションの名前は `p20220401_beijing` になります。

### 例

例 1: 時間範囲と特定の都市に基づいてデータセンターの請求詳細を頻繁にクエリする場合を想定します。テーブル作成時に、パーティション式を使用して最初のパーティション列を `dt` と `city` として指定できます。これにより、同じ日付と都市に属するデータが同じパーティションにルーティングされ、パーティションプルーニングを使用してクエリ効率を大幅に向上させることができます。

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

パーティションを確認します。結果は、StarRocksがロードされたデータに基づいて `p20220401_Houston` パーティションを自動的に作成したことを示しています。その後のロード中に、`dt` と `city` のパーティション列に `2022-04-01` と `Houston` の値を持つデータはこのパーティションに格納されます。

> **注記**
>
> 各パーティションは、パーティション列に指定された1つの値を持つデータのみを含むことができます。パーティション列に複数の値を指定するには、[リストパーティション](./list_partitioning.md)を参照してください。

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

例 2: テーブル作成時に `partition_live_number` プロパティを設定してパーティションのライフサイクル管理を行うこともできます。例えば、テーブルが最新の3つのパーティションのみを保持するように指定することができます。

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
    "partition_live_number" = "3" -- 最新の3つのパーティションのみを保持
);
```

## パーティションの管理

### パーティションへのデータロード

データロード中、StarRocksはロードされたデータとパーティション式によって定義されたパーティションルールに基づいて自動的にパーティションを作成します。

テーブル作成時に式パーティショニングを使用し、[INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select)を使って特定のパーティション内のデータを上書きする必要がある場合、パーティションが既に作成されているかどうかにかかわらず、現在は `PARTITION()` 内に明示的にパーティション範囲を提供する必要があります。これは、[レンジパーティショニング](./Data_distribution.md#range-partitioning)や[リストパーティショニング](./list_partitioning.md)と異なり、パーティション名のみを `PARTITION(<partition_name>)` で提供することができます。

テーブル作成時に時間関数式を使用し、特定のパーティションのデータを上書きしたい場合は、そのパーティションの開始日または日時（テーブル作成時に設定されたパーティションの粒度）を提供する必要があります。パーティションが存在しない場合、データロード中に自動的に作成されます。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

テーブル作成時に列式を使用し、特定のパーティションのデータを上書きしたい場合は、そのパーティションが含むパーティション列の値を提供する必要があります。パーティションが存在しない場合、データロード中に自動的に作成されます。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### パーティションの確認

自動的に作成されたパーティションに関する特定の情報を確認したい場合は、`SHOW PARTITIONS FROM <table_name>` ステートメントを使用する必要があります。`SHOW CREATE TABLE <table_name>` ステートメントは、テーブル作成時に設定された式パーティショニングの構文のみを返します。

```SQL
MySQL > SHOW PARTITIONS FROM t_recharge_detail1;
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName     | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | List                        | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 16890       | p20220401_Houston | 2              | 2023-07-19 17:24:53 | 0                  | NORMAL | dt, city     | (('2022-04-01', 'Houston')) | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
| 17056       | p20220402_texas   | 2              | 2023-07-19 17:27:42 | 0                  | NORMAL | dt, city     | (('2022-04-02', 'texas'))   | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

## 制限

- v3.1.0以降、StarRocksの[共有データモード](../deployment/shared_data/s3.md)は[時間関数式](#partitioning-based-on-a-time-function-expression)に基づくパーティショニングをサポートしています。また、v3.1.1以降、StarRocksの[共有データモード](../deployment/shared_data/s3.md)は[列式](#partitioning-based-on-the-column-expression-since-v31)に基づくパーティショニングをさらにサポートしています。
- 現在、CTASを使用して式に基づくパーティション設定がされたテーブルを作成することはサポートされていません。
- 現在、Spark Loadを使用して式に基づくパーティションを使用するテーブルにデータをロードすることはサポートされていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>` ステートメントを使用して列式によって作成されたパーティションを削除する場合、パーティション内のデータは直接削除され、復旧することはできません。
- 現在、式に基づくパーティショニングで作成されたパーティションの[バックアップおよびリストア](../administration/Backup_and_restore.md)はできません。
