---
displayed_sidebar: "Japanese"
---

# 式のパーティショニング（推奨）

v3.0から、StarRocksは式のパーティショニング（以前は自動パーティショニングと呼ばれていました）をサポートしており、これはより柔軟で使いやすいものです。このパーティショニング方法は、連続する時間範囲や列挙値に基づいてデータをクエリや管理するなどのほとんどのシナリオに適しています。

テーブル作成時に単純なパーティション式（日付関数式または列式のいずれか）を指定するだけで済みます。データのロード時、StarRocksはデータとパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。テーブル作成時に手動で多数のパーティションを作成する必要はなく、動的パーティションのプロパティを構成する必要もありません。

## 時間関数式に基づくパーティショニング

連続する時間範囲に基づいて頻繁にデータをクエリや管理する場合は、パーティション列として日付型（DATEまたはDATETIME）の列を指定し、時間関数式で年、月、日、または時間をパーティションの粒度として指定するだけです。StarRocksは自動的にパーティションを作成し、読み込まれたデータとパーティション式に基づいてパーティションの開始日時と終了日時または日付時刻を設定します。

ただし、過去のデータを月ごとのパーティションに、最近のデータを日ごとのパーティションに分割するような特殊なシナリオでは、[レンジ・パーティショニング](./Data_distribution.md#range-partitioning)を使用してパーティションを作成する必要があります。

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

| パラメータ | 必須    | 説明                                                                                       |
| ---------- | ------- | ------------------------------------------------------------------------------------------ |
| `expression`          | YES       | 現在、[date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md) および [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 関数のみサポートされています。`time_slice` 関数を使用する場合、`boundary` パラメータを渡す必要はありません。これは、このシナリオでは、このパラメータのデフォルトかつ有効な値が `floor` であり、値を `ceil` にすることはできないためです。 |
| `time_unit`        | YES       | パーティションの粒度。`hour`、 `day`、 `month`、または `year` であることが可能です。`week` パーティション粒度はサポートされていません。パーティション粒度が `hour` の場合、パーティション列はDATETIMEデータ型である必要があり、DATEデータ型であってはいけません。|
| `partition_column` | YES       | パーティション列の名前。<br/><ul><li>パーティション列はDATEまたはDATETIMEのデータ型である必要があります。パーティション列には `NULL` 値を許可できます。</li><li>パーティション列は `date_trunc` 関数を使用する場合、DATEまたはDATETIMEのデータ型であってはいけません。`time_slice` 関数を使用する場合、パーティション列はDATETIMEのデータ型である必要があります。</li><li>
パーティション列がDATEデータ型の場合、サポートされる範囲は【0000-01-01 ~ 9999-12-31】です。パーティション列がDATETIMEデータ型の場合、サポートされる範囲は【0000-01-01 01:01:01 ~ 9999-12-31 23:59:59】です。</li><li>現在、1つのパーティション列しか指定できず、複数のパーティション列はサポートされていません。</li></ul> |
| `partition_live_number` | NO        | 保持する最新のパーティションの数。ここで「最新」とは、パーティションが時間順にソートされ、**現在の日付を基準として**逆数されたパーティションの数を指し、それ以外のパーティション（より古いパーティション）は削除されます。StarRocksは、パーティションの数を管理するタスクをスケジュールし、スケジュール間隔はFE動的パラメータ `dynamic_partition_check_interval_seconds` で構成できます。このパラメータのデフォルト値は600秒（10分）です。たとえば、現在の日付が2023年4月4日であり、`partition_live_number` が `2` に設定されており、パーティションは `p20230401`、 `p20230402`、 `p20230403`、 `p20230404` が含まれています。パーティション `p20230403` と `p20230404` が保持され、その他のパーティション（はるかに前の時期に作成されたパーティション）は削除されます。不正なデータがロードされた場合、たとえば未来の日付である4月5日と4月6日のデータが含まれている場合、パーティションには `p20230401`、 `p20230402`、 `p20230403`、 `p20230404`、 `p20230405`、 `p20230406` が含まれます。その後、パーティション `p20230403`、 `p20230404`、 `p20230405`、 `p20230406` が保持され、その他のパーティションは削除されます。 |

### 使用上の注意

- データのロード時、StarRocksは読み込まれたデータに基づいて一部のパーティションを自動的に作成しますが、何らかの理由でロードジョブが失敗した場合、StarRocksによって自動的に作成されたパーティションは自動的に削除されません。
- StarRocksは、デフォルトで自動的に作成されるパーティションの最大数を4096として設定しており、これは手違いによる多数のパーティションの作成を防ぐことができます。
- パーティションの命名規則は、動的パーティショニングの命名規則と一貫しています。

### **例**

例1: 日ごとに頻繁にデータをクエリする場合。パーティション式として `date_trunc()` を使用し、テーブル作成時にパーティション列を `event_day` とし、パーティションの粒度を `day` と設定することができます。データは自動的に日付ごとにパーティション分割され、パーティションプルーニングを使用して問い合わせ効率を大幅に向上させることができます。

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

たとえば、次の2つのデータ行をロードすると、StarRocksは自動的に2つのパーティション `p20230226` と `p20230227` を作成し、それぞれの範囲を [2023-02-26 00:00:00, 2023-02-27 00:00:00) および [2023-02-27 00:00:00, 2023-02-28 00:00:00) として設定します。その後のデータがこれらの範囲に含まれる場合、それらは自動的に対応するパーティションにルーティングされます。

```SQL
-- 2つのデータ行を挿入
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

例2: 過去のパーティションを削除して最新のパーティションのみを保持するようにパーティションライフサイクル管理を実装したい場合は、`partition_live_number` プロパティを使用して保持するパーティションの数を指定できます。

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

Example 3: 週ごとにデータを頻繁にクエリする場合は、`time_slice()` のパーティション式を使用して、テーブル作成時にパーティション列を `event_day` に設定し、パーティションの粒度を7日に設定できます。1週間のデータが1つのパーティションに格納され、パーティションプルーニングを使用してクエリの効率が大幅に向上する。

```sql
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

## カラム式に基づくパーティショニング（v3.1以降）

特定のタイプのデータを頻繁にクエリおよび管理する場合、そのタイプを表す列をパーティション列として指定するだけでよいです。StarRocksは、読み込まれたデータのパーティション列の値に基づいて自動的にパーティションを作成します。

ただし、テーブルに `city` などの列が含まれ、国と都市に基づいて頻繁にクエリおよびデータを管理する場合など、[list partitioning](./list_partitioning.md) を使用して、同じ国内の複数の都市のデータを1つのパーティションに格納する必要があります。

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

| **パラメーター** | **必須** | **説明** |
| ----------------------------- | -------- | ----------------------------------------------------- |
| `partition_columns` | YES | パーティション列の名前。<br/> <ul><li>パーティション列の値は文字列（BINARYはサポートされていません）、日付または日時、整数、およびブール値にできます。パーティション列には `NULL` 値を許容します。</li><li>各パーティションには、パーティション列の値が同じデータのみを含めることができます。パーティション内のパーティション列の異なる値を含める場合は、[List partitioning](./list_partitioning.md) を参照してください。</li></ul> |
| `partition_live_number` | No | 保持するパーティションの数です。パーティション間でパーティション列の値を比較し、より小さい値を持つパーティションを定期的に削除し、より大きい値を持つパーティションを保持します。<br/>StarRocksは、パーティションの数を管理するためのタスクをスケジュールし、スケジュールされる間隔はFE動的パラメータ `dynamic_partition_check_interval_seconds` を介して設定できます。デフォルトは600秒（10分）です。<br/>**注意**<br/>パーティション列の値が文字列の場合、StarRocksはパーティション名の辞書順を比較し、後続のパーティションを削除しながら、先行するパーティションを定期的に保持します。 |

### 使用上の注意

- データの読み込み中、StarRocksは読み込まれたデータに基づいて一部のパーティションを自動的に作成しますが、何らかの理由で読み込みジョブに失敗すると、StarRocksによって自動的に作成されたパーティションは自動的に削除されません。
- StarRocksは、デフォルトで自動的に作成されるパーティションの最大数を4096と設定しています。誤って多くのパーティションを作成することを防ぐためのパラメータです。
- パーティションの命名規則: 複数のパーティション列が指定されている場合、異なるパーティション列の値は、パーティション名でアンダースコア `_` で接続され、フォーマットは `p<パーティション列1の値>_<パーティション列2の値>_...` です。 例えば、`dt` と `province` の2つの列がパーティション列として指定されており、どちらも文字列型であり、値が `2022-04-01` および `beijing` のデータ行が読み込まれた場合、自動的に作成される対応するパーティションの名前は `p20220401_beijing` となります。

### 例

例1: データセンターの請求の詳細を時間範囲と特定の都市に基づいて頻繁にクエリする場合は、テーブル作成時にパーティション式を使用して、最初のパーティション列を `dt` および `city` として指定できます。これにより、同じ日付および都市に属するデータが同じパーティションにルーティングされ、パーティションプルーニングを使用してクエリの効率が大幅に向上します。

```sql
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

```sql
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

パーティションを表示します。結果は、StarRocksが読み込まれたデータに基づいて自動的にパーティション `p20220401_Houston1` を作成したことを示します。その後の読み込み中、`dt` および `city` のパーティション列で値が `2022-04-01` および `Houston` のデータがこのパーティションに格納されます。

> **注意**
>
> 各パーティションには、指定されたパーティション列の値が1つだけ含まれます。パーティション列の1つ以上の値をパーティションに指定する場合は、[List partitions](./list_partitioning.md) を参照してください。

```sql
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

例2: テーブル作成時に "`partition_live_number`プロパティを指定してパーティションのライフサイクルを管理することもできます。たとえば、テーブルが最新の3つのパーティションのみを保持するように指定できます。

```sql
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

### パーティションへのデータの読み込み

データの読み込み中、StarRocksは読み込まれたデータとパーティション式に基づいて自動的にパーティションを作成します。

テーブル作成時に式パーティショニングを使用し、特定のパーティションでデータを上書きする場合、パーティションが作成されているかどうかに関係なく、`PARTITION（）` で明示的にパーティション範囲を提供する必要があります。これは[Range Partitioning](./Data_distribution.md#range-partitioning) または[List Partitioning](./list_partitioning.md) の場合と異なり、`PARTITION（<partition_name>）` のみでパーティション名を提供できます。

テーブル作成時に時間関数式を使用し、特定のパーティションでデータを上書きする場合、そのパーティションの開始日またはdatetime（テーブル作成時に設定したパーティションの粒度）を提供する必要があります。パーティションが存在しない場合、読み込み中に自動的に作成できます。

```sql
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

テーブル作成時に列式を使用し、特定のパーティションでデータを上書きする場合、パーティションが含むパーティション列の値を提供する必要があります。パーティションが存在しない場合、読み込み中に自動的に作成できます。

```sql
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### パーティションの表示

自動的に作成されたパーティションに関する特定の情報を表示する場合は、`SHOW PARTITIONS FROM <table_name>` ステートメントを使用する必要があります。`SHOW CREATE TABLE <table_name>` ステートメントは、テーブル作成時に構成された式パーティショニングの構文のみを返します。
| PartitionId | PartitionName     | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | List                        | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 16890       | p20220401_ヒューストン | 2              | 2023-07-19 17:24:53 | 0                  | 正常   | dt, city     | (('2022-04-01', 'ヒューストン')) | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
| 17056       | p20220402_テキサス   | 2              | 2023-07-19 17:27:42 | 0                  | 正常   | dt, city     | (('2022-04-02', 'テキサス'))   | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

## 制限

- v3.1.0以降、StarRocksの[共有データモード](../deployment/shared_data/s3.md)は[時間関数式](#partitioning-based-on-a-time-function-expression)をサポートしています。そしてv3.1.1以降、StarRocksの[共有データモード](../deployment/shared_data/s3.md)はさらに[列式](#partitioning-based-on-the-column-expression-since-v31)をサポートしています。
- 現在、CTASを使用して表を作成する際に構成した式のパーティショニングはサポートされていません。
- 現在、式のパーティショニングを使用しているテーブルにデータをロードする際にSpark Loadを使用することはサポートされていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>` ステートメントを使用して列式を使用して作成したパーティションを削除すると、パーティション内のデータが直接削除され、復元されません。
- 現在、式によって作成されたパーティションの[バックアップとリストア](../administration/Backup_and_restore.md)はできません。