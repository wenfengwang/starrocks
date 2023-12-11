---
displayed_sidebar: "Japanese"
---

# リストパーティション

v3.1以降、StarRocksはリストパーティションをサポートしています。データは各パーティションの事前定義値リストに基づいてパーティショニングされ、クエリの高速化や列挙された値に基づいた管理を容易にすることができます。

## 導入

各パーティションで列の値リストを明示的に指定する必要があります。これらの値は、範囲パーティショニングで必要な連続した時間や数値範囲とは異なります。データのロード中、StarRocksはデータのパーティショニング列の値と各パーティションの事前定義列の値との間のマッピングに基づいてデータを対応するパーティションに保存します。

![list_partitioning](../assets/list_partitioning.png)

リストパーティショニングは、列に少数のenum値を含むデータを保存するのに適しており、通常、これらのenum値に基づいてクエリやデータ管理を行います。例えば、列が地理的位置、州、カテゴリを表している場合です。列の各値は独立したカテゴリを表します。列挙値に基づいてデータをパーティション化することで、クエリのパフォーマンスを向上させ、データ管理を容易にすることができます。

**リストパーティショニングは、1つのパーティションに複数の値を含める必要があるシナリオに特に有用です**。例えば、個人の出身地を表す`City`列を含むテーブルがあり、州と都市に基づいて頻繁にクエリやデータ管理を行う場合です。テーブル作成時、`City`列をリストパーティショニングのパーティション列として使用し、同じ州内のさまざまな都市のデータを1つのパーティションに配置するよう指定することができます。例えば、`PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。この方法により、州と都市に基づいたクエリの高速化やデータ管理を容易にすることができます。

パーティションにはパーティショニング列の各値と同じ値のデータのみを含める必要がある場合、[式パーティショニング](./expression_partitioning.md)を使用することをお勧めします。

**リストパーティショニングと[式パーティショニング](./expression_partitioning.md)の比較**

リストパーティショニングと式パーティショニング（推奨）の主な違いは、リストパーティショニングではパーティションを手動で1つずつ作成する必要がある点です。一方、式パーティショニングではデータのロード中にパーティションを自動的に作成することができ、パーティショニングが簡素化されます。ほとんどの場合、式パーティショニングでリストパーティショニングを代替することができます。2つの具体的な比較は以下の表に示されています。

| パーティション方法                                  | **リストパーティショニング**                                | **式パーティショニング**                                      |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 構文                                                | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                           |
| パーティショニング列の各値についての複数の値の対応 | サポートされています。1つのパーティションには、パーティショニング列ごとに異なる値のデータが格納されることができます。次の例では、ロードされたデータに`city`列の`Los Angeles`、`San Francisco`、`San Diego`の値が含まれる場合、すべてのデータが1つのパーティションに格納されます。`pCalifornia`.`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | サポートされていません。1つのパーティションにはパーティショニング列の同じ値のデータが格納されます。例えば、式パーティショニングでは`PARTITION BY (city)`が使用されます。`city`列の`Los Angeles`、`San Francisco`、`San Diego`の値が含まれるデータをロードすると、StarRocksは自動的に`pLosAngeles`、`pSanFrancisco`、`pSanDiego`の3つのパーティションを作成します。3つのパーティションはそれぞれ、`city`列の`Los Angeles`、`San Francisco`、`San Diego`の値を持つデータを格納します。 |
| データロード前のパーティションの作成                | サポートされています。テーブル作成時にパーティションを作成する必要があります。 | 不要です。データのロード中にパーティションが自動的に作成されることがあります。 |
| データロード中にリストパーティションを自動的に作成 | サポートされていません。データのロード中に対応するパーティションが存在しない場合、エラーが返されます。 | サポートされています。データのロード中に対応するパーティションが存在しない場合、StarRocksはデータを格納するためにパーティションを自動的に作成します。各パーティションは、パーティショニング列の同じ値のデータのみを含むことができます。 |
| SHOW CREATE TABLE                                    | CREATE TABLE ステートメント内でパーティションの定義が返されます。 | データがロードされた後、ステートメントはCREATE TABLE ステートメントで使用されたパーティション句を返します、つまり、`PARTITION BY (partition_columns)`です。しかし、返された結果には自動的に作成されたパーティションは表示されません。自動的に作成されたパーティションを表示するには、`SHOW PARTITIONS FROM <table_name>`を実行する必要があります。 |

## 使用法

### 構文

```sql
PARTITION BY LIST (partition_columns)（
    PARTITION <partition_name> VALUES IN (value_list)
    [, ...]
)

partition_columns::= 
    <column> [,<column> [, ...] ]

value_list ::=
    value_item [, value_item [, ...] ]

value_item ::=
    { <value> | ( <value> [, <value>, [, ...] ] ) }    
```

### パラメータ

| **パラメータ**    | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| `partition_columns` | YES      | パーティショニング列の名前です。パーティショニング列の値は文字列（BINARYは非対応）、日付または日時、整数、ブール値にすることができます。パーティショニング列は`NULL`値を許容します。 |
| `partition_name`  | YES      | パーティション名。異なるパーティション内のデータを区別するために、ビジネスシナリオに基づいた適切なパーティション名を設定することがお勧めです。 |
| `value_list`      | YES      | パーティショニング列のパーティション内の値のリスト。              |

### 例

例1：データセンターの請求の詳細を州や都市ごとに頻繁にクエリする場合を想定します。テーブル作成時、パーティション列を`city`とし、各パーティションが同じ州内の都市データを格納するよう指定することができます。この方法により、特定の州や都市のクエリを高速化し、データ管理を容易にすることができます。

```SQL
CREATE TABLE t_recharge_detail1 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY LIST (city) (
   PARTITION pLos_Angeles VALUES IN ("Los Angeles"),
   PARTITION pSan_Francisco VALUES IN ("San Francisco")
)
DISTRIBUTED BY HASH(`id`);
```

例2：時間の範囲と特定の州や都市に基づいてデータセンターの請求の詳細を頻繁にクエリする場合を想定します。テーブル作成時、パーティション列を`dt`と`city`とし、特定の日付と特定の州や都市のデータが同じパーティションに格納されるよう指定することができます。これにより、クエリ速度が向上し、データ管理が容易になります。

```SQL
CREATE TABLE t_recharge_detail4 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
) ENGINE=OLAP
DUPLICATE KEY(id)
PARTITION BY LIST (dt,city) (
   PARTITION p202204_California VALUES IN (
       ("2022-04-01", "Los Angeles"),
       ("2022-04-01", "San Francisco"),
       ("2022-04-02", "Los Angeles"),
       ("2022-04-02", "San Francisco")
    ),
   PARTITION p202204_Texas VALUES IN (
       ("2022-04-01", "Houston"),
       ("2022-04-01", "Dallas"),
       ("2022-04-02", "Houston"),
       ("2022-04-02", "Dallas")
   )
)
DISTRIBUTED BY HASH(`id`);
```

## 制限

- リストパーティショニングはダイナミックパーティショニングや一度に複数のパーティションを作成する機能をサポートしていません。
- 現在、StarRocksの[共有データモード](../deployment/shared_data/s3.md)はこの機能をサポートしていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` ステートメントを使用してリストパーティショニングを使用して作成したパーティションを削除すると、パーティション内のデータが直接削除され、回復することはできません。
- 現在、リストパーティションで作成されたパーティションを[バックアップおよび復元](../administration/Backup_and_restore.md)することはできません。
- 現在、StarRocksはリストパーティショニングの戦略で作成された[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)の作成をサポートしていません。