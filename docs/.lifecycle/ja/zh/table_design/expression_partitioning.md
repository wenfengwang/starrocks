---
displayed_sidebar: Chinese
---

# 式分割（推奨）

v3.0 以降、StarRocks は式分割（旧称自動作成分割）をサポートしており、より柔軟で使いやすく、連続した日付範囲や列挙値によるデータのクエリや管理に適しています。

テーブルを作成する際には、分割式（時間関数式またはリスト式）を設定するだけで済みます。データのインポート時には、StarRocks がデータと分割式の定義ルールに基づいて自動的に分割を作成するため、テーブル作成時にあらかじめ手動で大量の分割を作成したり、動的分割属性を設定する必要はありません。

## 時間関数式による分割

連続した日付範囲でデータのクエリや管理を頻繁に行う場合は、時間関数式の分割式で日付型（DATE または DATETIME）の分割列を指定し、分割の粒度（年、月、日、または時間）を設定するだけで済みます。StarRocks はインポートされたデータと分割式に基づいて自動的に分割を作成し、分割の開始と終了の時間を設定します。

ただし、特定のシナリオでは、例えば過去のデータを月単位で分割したり、最近のデータを日単位で分割したりする場合は、[Range 分割](./Data_distribution.md#range-分割)を使用して分割を作成する必要があります。

### 構文

```sql
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    { date_trunc ( <time_unit> , <partition_column> ) |
      time_slice ( <partition_column> , INTERVAL <N> <time_unit> [ , boundary ] ) }
```

### パラメーター説明

| パラメーター              | 必須 | 説明                                                         |
| ----------------------- | ---- | ------------------------------------------------------------ |
| `expression`            | はい | 現在、[date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md) と [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 関数のみをサポートしています。また、`time_slice` 関数を使用する場合は、`boundary` パラメーターを省略できます。このシナリオでは、このパラメーターはデフォルトで `floor` のみをサポートし、`ceil` はサポートしていません。 |
| `time_unit`             | はい | 分割の粒度で、現在は `hour`、`day`、`month`、`year` のみをサポートしており、`week` はサポートしていません。分割の粒度が `hour` の場合は、分割列は DATETIME 型のみをサポートし、DATE 型はサポートしていません。 |
| `partition_column`      | はい | 分割列。 <br /> <ul><li>日付型（DATE または DATETIME）のみをサポートし、他の型はサポートしていません。`date_trunc` 関数を使用する場合は、分割列は DATE または DATETIME 型をサポートします。`time_slice` 関数を使用する場合は、分割列は DATETIME 型のみをサポートします。分割列の値は `NULL` をサポートします。</li><li> 分割列が DATE 型の場合は、範囲は [0000-01-01 ~ 9999-12-31] をサポートします。DATETIME 型の場合は、範囲は [0000-01-01 01:01:01 ~ 9999-12-31 23:59:59] をサポートします。</li><li> 現在は1つの分割列のみを指定でき、複数の分割列を指定することはできません。</li></ul> |
| `partition_live_number` | いいえ | 保持する最新の分割数。最新とは、分割が時間の順序で並べられ、**現在の時間**を基準に、後ろから指定された数の分割を保持し、それ以前の分割は削除されることを意味します。バックグラウンドで定期的に分割数を管理するタスクがスケジュールされ、その間隔は FE の動的パラメーター `dynamic_partition_check_interval_seconds` で設定でき、デフォルトは 600 秒、つまり 10 分です。例えば、現在が 2023 年 4 月 4 日で、`partition_live_number` が `2` に設定されている場合、分割に `p20230401`、`p20230402`、`p20230403`、`p20230404` が含まれていると、`p20230403`、`p20230404` が保持され、他の分割は削除されます。未来の日付である 4 月 5 日と 6 日のデータがインポートされ、分割に `p20230401`、`p20230402`、`p20230403`、`p20230404`、`p20230405`、`p20230406` が含まれている場合は、`p20230403`、`p20230404`、`p20230405`、`p20230406` が保持され、他の分割は削除されます。|

### 使用上の注意

- データのインポート中に StarRocks が自動的にいくつかの分割を作成しましたが、何らかの理由でインポートジョブが最終的に失敗した場合、現在のバージョンでは、自動的に作成された分割はインポート失敗によって自動的に削除されません。
- StarRocks が自動的に作成する分割の最大数はデフォルトで 4096 で、FE の設定パラメーター `max_automatic_partition_number` によって決まります。このパラメーターは、誤操作による大量の分割の作成を防ぐためのものです。
- 分割の命名規則は動的分割の命名規則と一致しています。

### 例

例 1：日ごとにデータをクエリすることが多い場合、テーブル作成時に `date_trunc()` の分割式を使用し、分割列を `event_day` とし、分割の粒度を `day` に設定することで、インポートされたデータが自動的に所属する日付に基づいて分割されます。同じ日のデータを1つの分割に格納することで、分割のプルーニングを利用してクエリの効率を大幅に向上させることができます。

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

以下の2行のデータをインポートすると、StarRocks はインポートされたデータの日付範囲に基づいて自動的に2つの分割 `p20230226` と `p20230227` を作成し、それぞれの範囲は [2023-02-26 00:00:00, 2023-02-27 00:00:00) と [2023-02-27 00:00:00, 2023-02-28 00:00:00) です。後続のデータがこれらの範囲に属している場合は、自動的に対応する分割に分類されます。

```SQL
-- 2行のデータをインポート
INSERT INTO site_access1 
    VALUES ("2023-02-26 20:12:04",002,"New York","Sam Smith",1),
           ("2023-02-27 21:06:54",001,"Los Angeles","Taylor Swift",1);

-- 分割をクエリ
mysql > SHOW PARTITIONS FROM site_access1;
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range                                                                                                | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 17138       | p20230226     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-26 00:00:00]; ..types: [DATETIME]; keys: [2023-02-27 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
| 17113       | p20230227     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-27 00:00:00]; ..types: [DATETIME]; keys: [2023-02-28 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

例 2：分割のライフサイクル管理を導入し、最近の分割のみを保持して歴史的な分割を削除したい場合は、`partition_live_number` を使用して保持する最新の分割数を設定できます。

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
    "partition_live_number" = "3" -- 最近の 3 つの分割のみを保持
);
```

例 3：週単位でデータをクエリすることが多い場合、テーブル作成時に `time_slice()` の分割式を使用し、分割列を `event_day` とし、分割の粒度を7日間に設定することで、1週間のデータを1つの分割に格納します。これにより、分割のプルーニングを利用してクエリの効率を大幅に向上させることができます。

```SQL
CREATE TABLE site_access3 (
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)

DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY time_slice(event_day, INTERVAL 7 day)
DISTRIBUTED BY HASH(event_day, site_id);
```

## リスト式パーティショニング（v3.1から）

特定の列挙値に基づいてデータを頻繁にクエリや管理する場合は、その型を表す列をパーティション列として指定するだけで、StarRocksはインポートされたデータのパーティション列の値に基づいて自動的にパーティションを分割し、作成します。

ただし、表に都市を表す列が含まれているなど、特定のシナリオでは、国と都市に基づいてデータをクエリや管理することが多く、同じ国に属する複数の都市のデータを1つのパーティションに格納したい場合は、[リストパーティショニング](./list_partitioning.md)を使用する必要があります。

### 構文

```bnf
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    ( <partition_columns> )
    
partition_columns ::=
    <column>, [ <column> [,...] ]
```

### パラメータ説明

| パラメータ                  | 必須     | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| `partition_columns`         | はい     | パーティション列。<br /><ul><li>文字列（BINARY型は不可）、日付、整数、ブール値がサポートされています。パーティション列の値が `NULL` であることはサポートされていません。</li><li>インポート後に自動的に作成されたパーティションには、それぞれのパーティション列の1つの値のみを含めることができます。複数の値を含めるには、[リストパーティショニング](./list_partitioning.md)を使用してください。</li></ul> |
| `partition_live_number`     | いいえ   | 保持するパーティションの数。これらのパーティションが含む値を比較し、定期的に小さい値のパーティションを削除し、大きい値のものを保持します。バックグラウンドで定期的にパーティション数を管理するタスクがスケジュールされ、その間隔は FE の動的パラメータ `dynamic_partition_check_interval_seconds` で設定でき、デフォルトは600秒、つまり10分です。<br />**注記**<br />パーティション列が文字列型の場合、パーティション名の辞書順に基づいて定期的に前にあるパーティションを保持し、後ろにあるパーティションを削除します。 |

### 使用説明

- インポートプロセス中にStarRocksはデータに基づいて自動的にいくつかのパーティションを作成しますが、何らかの理由でインポートジョブが最終的に失敗した場合、現在のバージョンでは、自動的に作成されたパーティションはインポート失敗によって自動的に削除されません。
- StarRocksで自動的に作成されるパーティションの最大数はデフォルトで4096で、FEの設定パラメータ `max_automatic_partition_number` によって決まります。このパラメータは、誤操作によって大量のパーティションを作成することを防ぐためのものです。
- パーティションの命名規則：複数のパーティション列が存在する場合、異なるパーティション列の値はアンダースコア（_）で接続されます。例えば、`dt` と `city` の2つのパーティション列があり、両方とも文字列型で、`2022-04-01`, `beijing` のデータをインポートすると、自動的に作成されるパーティション名は `p20220401_beijing` になります。

### 例

例1：日付範囲と特定の都市に基づいてデータセンターの料金詳細を頻繁にクエリする場合、テーブル作成時にパーティション式を使用して、日付 `dt` と都市 `city` をパーティション列として指定することができます。これにより、同じ日付と都市に属するデータが同じパーティションにグループ化され、パーティションプルーニングを使用することでクエリの効率が大幅に向上します。

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

データをインポートします。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

具体的なパーティションを確認します。結果は、StarRocksがインポートデータのパーティション列の値に基づいて自動的にパーティション `p20220401_Houston` を作成したことを示しています。後続のインポートデータのパーティション列 `dt` と `city` の値が `2022-04-01` と `Houston` であれば、それらはすべてこのパーティションに割り当てられます。

> **注記**
>
> パーティションには、それぞれのパーティション列の1つの値のみを含めることができます。複数の値を含めるには、[リストパーティショニング](./list_partitioning.md)を使用してください。

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

例2：テーブル作成時に `partition_live_number` パラメータを設定してパーティションのライフサイクルを管理することもできます。例えば、最新の3つのパーティションのみを保持するように指定します。

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
    "partition_live_number" = "3" -- 最新の3つのパーティションのみを保持します。
);
```

## パーティションの管理

### パーティションへのデータのインポート

データをインポートする際、StarRocksはデータとパーティション式の定義に基づいて自動的にパーティションを作成します。

注目すべき点は、表式パーティショニングを使用してテーブルを作成し、[INSERT OVERWRITE](../loading/InsertInto.md#insert-overwrite-select文を使用したデータの上書き)を使用して**特定のパーティション**のデータを上書きする必要がある場合、そのパーティションが既に作成されているかどうかに関わらず、`PARTITION()`内で明確なパーティション範囲を提供する必要があります。これは、RangeパーティショニングやListパーティショニングとは異なり、`PARTITION (partition_name)`でパーティション名を提供するだけでは不十分です。

時間関数式パーティショニングを使用してテーブルを作成した場合、特定のパーティションにデータを上書きする際には、そのパーティションの開始範囲（パーティションの粒度はテーブル作成時に設定した粒度と一致する必要があります）を提供する必要があります。そのパーティションが存在しない場合、データをインポートする際に自動的に作成されます。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

リスト式パーティショニングを使用してテーブルを作成した場合、特定のパーティションにデータを上書きする際には、そのパーティションが含むパーティション列の値を提供する必要があります。そのパーティションが存在しない場合、データをインポートする際に自動的に作成されます。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### パーティションの確認

自動的に作成されたパーティションの詳細情報を確認するには、`SHOW PARTITIONS FROM <table_name>`文を使用する必要があります。一方、`SHOW CREATE TABLE <table_name>`文の結果には、テーブル作成時に設定されたパーティション式の構文のみが含まれます。

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

## 使用制限

- v3.1.0から、StarRocksの[計算ストレージ分離モード](../deployment/shared_data/s3.md)は[時間関数式パーティショニング](#時間関数式パーティショニング)をサポートしています。また、v3.1.1からはリスト式パーティショニングもサポートしています。
- CTASを使用したテーブル作成では、現時点ではパーティション式はサポートされていません。
- Spark Loadを使用してパーティション式のテーブルにデータをインポートすることは、現時点ではサポートされていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>` を使用してリスト式パーティションを削除する場合、パーティションは直接削除され、復元することはできません。
- リスト式パーティションは現在、[バックアップとリストア](../administration/Backup_and_restore.md)をサポートしていません。
- リスト式パーティションを使用する場合、2.5.4以降のバージョンにのみロールバックをサポートします。
