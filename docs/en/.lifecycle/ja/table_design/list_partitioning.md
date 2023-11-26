---
displayed_sidebar: "Japanese"
---

# リストパーティショニング

StarRocksはv3.1以降、リストパーティショニングをサポートしています。データは、各パーティションごとに事前定義された値リストに基づいてパーティション分割され、列挙値に基づいたクエリの高速化と管理の容易化が可能です。

## 概要

各パーティションで列の値リストを明示的に指定する必要があります。これらの値は、連続した時間や数値範囲が必要なRange Partitioningとは異なり、連続性を持たずに指定することができます。データのロード時、StarRocksはデータのパーティション列の値と各パーティションの事前定義された列の値とのマッピングに基づいて、データを対応するパーティションに格納します。

![list_partitioning](../assets/list_partitioning.png)

リストパーティショニングは、列に少数の列挙値を含むデータを格納するのに適しており、これらの列挙値に基づいてデータをクエリや管理することが頻繁に行われる場合に使用します。例えば、列が地理的な場所、州、カテゴリを表す場合です。列の各値は独立したカテゴリを表します。列挙値に基づいてデータをパーティション分割することで、クエリのパフォーマンスを向上させ、データの管理を容易にすることができます。

**リストパーティショニングは、パーティションごとに複数の値を含める必要があるシナリオに特に有用です**。例えば、個人の出身地を表す`City`列を含むテーブルがあり、州と都市に基づいてデータを頻繁にクエリや管理する場合です。テーブル作成時に、`City`列をリストパーティショニングのパーティション列として使用し、同じ州内のさまざまな都市のデータを1つのパーティションに配置するように指定することができます。例えば、`PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`とします。この方法により、州と都市に基づいたクエリを高速化し、データの管理を容易にすることができます。

パーティションごとにパーティショニング列の値が同じであるデータのみを含める必要がある場合は、[式パーティショニング](./expression_partitioning.md)を使用することをおすすめします。

**リストパーティショニングと[式パーティショニング](./expression_partitioning.md)の比較**

リストパーティショニングと式パーティショニング（推奨）の主な違いは、リストパーティショニングではパーティションを手動で1つずつ作成する必要がある点です。一方、式パーティショニングではデータのロード時にパーティションを自動的に作成することができ、パーティショニングを簡素化することができます。また、ほとんどの場合、式パーティショニングはリストパーティショニングを置き換えることができます。具体的な比較は以下の表に示されています。

| パーティショニング方法                             | **リストパーティショニング**                                       | **式パーティショニング**                                         |
| --------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------- |
| 構文                                                | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                                  |
| パーティションごとのパーティショニング列の複数の値 | サポートされています。パーティションごとに異なる値を含むデータをパーティションに格納することができます。次の例では、ロードされたデータが`city`列に`Los Angeles`、`San Francisco`、`San Diego`の値を含む場合、すべてのデータが1つのパーティションに格納されます。`pCalifornia`.`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | サポートされていません。パーティションは、パーティショニング列の同じ値を持つデータを格納します。例えば、式`PARTITION BY (city)`は式パーティショニングで使用されます。ロードされたデータが`city`列に`Los Angeles`、`San Francisco`、`San Diego`の値を含む場合、StarRocksは自動的に`pLosAngeles`、`pSanFrancisco`、`pSanDiego`の3つのパーティションを作成します。3つのパーティションはそれぞれ、`city`列の値が`Los Angeles`、`San Francisco`、`San Diego`であるデータを格納します。 |
| データロード前のパーティションの作成                 | サポートされています。テーブル作成時にパーティションを作成する必要があります。 | 作成する必要はありません。データのロード時にパーティションが自動的に作成されることがあります。 |
| データロード時にリストパーティションを自動作成する   | サポートされていません。データロード時に対応するパーティションが存在しない場合、エラーが返されます。 | サポートされています。データロード時に対応するパーティションが存在しない場合、StarRocksはデータを格納するためにパーティションを自動的に作成します。各パーティションは、パーティショニング列の同じ値を持つデータのみを含むことができます。 |
| SHOW CREATE TABLE                                   | CREATE TABLEステートメントでパーティションの定義を返します。 | データがロードされた後、ステートメントはCREATE TABLEステートメントで使用されたパーティション句を返します。ただし、返される結果には自動的に作成されたパーティションは表示されません。自動的に作成されたパーティションを表示する必要がある場合は、`SHOW PARTITIONS FROM <table_name>`を実行してください。 |

## 使用方法

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

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------------- | -------- | ------------------------------------------------------------ |
| `partition_columns` | YES      | パーティション化する列の名前。パーティション化列の値は、文字列（BINARYはサポートされていません）、日付または日時、整数、およびブール値にすることができます。パーティション化列は`NULL`値を許容します。 |
| `partition_name`    | YES      | パーティション名。ビジネスシナリオに基づいて適切なパーティション名を設定し、異なるパーティションのデータを区別することをおすすめします。 |
| `value_list`        | YES      | パーティション内のパーティショニング列の値のリスト。            |

### 例

例1: データセンターの請求の詳細を州または都市に基づいて頻繁にクエリする場合を想定します。テーブル作成時に、パーティショニング列を`city`として指定し、同じ州内の都市のデータを各パーティションに格納するように指定することができます。この方法により、特定の州や都市に対するクエリを高速化し、データの管理を容易にすることができます。

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

例2: データセンターの請求の詳細を時間範囲と特定の州または都市に基づいて頻繁にクエリする場合を想定します。テーブル作成時に、パーティショニング列を`dt`と`city`として指定することで、特定の日付と特定の州または都市のデータを同じパーティションに格納することができます。これにより、クエリの速度が向上し、データの管理が容易になります。

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

## 制限事項

- リストパーティショニングは、動的パーティショニングや一度に複数のパーティションの作成をサポートしています。
- 現在、StarRocksの[共有データモード](../deployment/shared_data/s3.md)はこの機能をサポートしていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>;`ステートメントを使用してリストパーティショニングで作成されたパーティションを削除する場合、パーティション内のデータは直接削除され、復元することはできません。
- 現在、リストパーティショニングで作成されたパーティションを[バックアップとリストア](../administration/Backup_and_restore.md)することはできません。
- 現在、リストパーティショニングのストラテジーで作成されたベーステーブルを使用して[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)を作成することはできません。
