```markdown
---
displayed_sidebar： "日本語"
---

# リストパーディション

v3.1以降、StarRocksはリストパーディションをサポートしています。データは、各パーディションの事前定義値リストに基づいてパーティション化されます。これにより、列挙された値に応じてクエリを高速化し、管理を容易にすることができます。

## はじめに

各パーティションで列の値リストを明示的に指定する必要があります。これらの値は連続する必要はなく、Range Partitioningで必要とされるような連続した時刻や数値範囲と異なります。StarRocksはデータをロードする際、データのパーディション列の値と各パーティションの事前に定義された列の値とのマッピングに基づいて、対応するパーティションにデータを格納します。

![list_partitioning](../assets/list_partitioning.png)

リストパーディションは、列に少数のenum値を含むデータを格納するのに適しており、これらのenum値に基づいてデータを頻繁にクエリおよび管理する場合に使用されます。たとえば、列が地理的位置、州、カテゴリを表す場合に使用されます。列の各値は独立したカテゴリを表します。enum値に基づいてデータをパーティショニングすることで、クエリのパフォーマンスを向上させ、データの管理を容易にすることができます。

**リストパーディションは、各パーティションに複数の値を含める必要があるシナリオに特に有用です**。たとえば、テーブルには個人の出身地を表す「City」列が含まれており、頻繁に州や都市単位でデータをクエリおよび管理する場合に使用されます。テーブルの作成時には、「City」列をリストパーディションのパーディション列として使用し、同じ州内のさまざまな都市のデータを1つのパーティションに配置するように指定できます。例えば`PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`のようにすることで、州と都市に基づいたクエリを高速化し、データの管理を容易にすることができます。

もしパーティションに含まれるデータがパーティション化列の同じ値を持つだけでよい場合は、[式パーディション](./expression_partitioning.md)を使用することが推奨されます。

**リストパーディションと[式パーディション](./expression_partitioning.md)の比較**

リストパーディションと式パーディション（推奨）の主な違いは、リストパーディションでは手動でパーティションを1つずつ作成する必要がある点です。一方、式パーディションではデータロード中にパーティションを自動的に作成できるため、パーティションの作成を簡素化できます。ほとんどのケースで、式パーディションがリストパーディションを置き換えることができます。両者の具体的な比較は以下の表に示されています：

| パーディショニング方法       | **リストパーディション**                                        | **式パーディション**                                           |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 構文                         | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                           |
| パーディショニング列毎のパーティション内の複数の値 | サポートされています。パーティションごとに異なる値を持つデータを格納できます。次の例では、ロードされたデータが`city`列に`Los Angeles`、`San Francisco`、および`San Diego`の値を含む場合、すべてのデータが1つのパーティションに格納されます。`pCalifornia`.`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | サポートされていません。パーティションは、パーティショニング列の同じ値を持つデータのみを格納します。例えば、式パーディショニングでは `PARTITION BY (city)`が使用されます。`city`列に`Los Angeles`、`San Francisco`、および`San Diego`の値を含む場合、StarRocksは`pLosAngeles`、`pSanFrancisco`、`pSanDiego`の3つのパーティションを自動的に作成します。それぞれのパーティションには、`city`列に`Los Angeles,`、`San Francisco`、および`San Diego`の値を持つデータが格納されます。 |
| データロード前のパーティション作成 | サポートされています。テーブル作成時にパーティションを作成する必要があります。  | そのような作業は不要です。データロード中にパーティションが自動的に作成されます。 |
| データロード中にリストパーティションを自動作成 | サポートされていません。データロード中に対応するパーティションが存在しない場合、エラーが返されます。 | サポートされています。データロード中に対応するパーティションが存在しない場合、StarRocksはデータを格納するためにパーティションを自動的に作成します。各パーティションにはパーティショニング列の同じ値のデータのみを含めることができます。 |
| SHOW CREATE TABLE            | CREATE TABLEステートメントでパーティションの定義を返します。 | データがロードされた後、ステートメントはCREATE TABLEステートメントで使用されたパーティション句を返します。つまり、`PARTITION BY (partition_columns)`です。ただし、自動的に作成されたパーティションは表示されません。自動的に作成されたパーティションを表示する必要がある場合は、`SHOW PARTITIONS FROM <table_name>`を実行してください。 |

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

| パラメータ     | 必須   | 説明                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| `partition_columns` | YES    | パーティション列の名前。パーティション列の値は、string（BINARYはサポートされていません）、dateまたはdatetime、integer、boolean値を許可します。パーティション列は`NULL`値を許可します。 |
| `partition_name` | YES    | パーティション名。異なるパーティション内のデータを区別するために、ビジネスシナリオに基づいて適切なパーティション名を設定することをお勧めします。 |
| `value_list`     | YES    | パーティション内の列の値のリスト。                             |

### 例

例1：データセンターの課金の詳細を州や都市別に頻繁にクエリするケースを考えます。テーブル作成時に、`city`をパーティション列として指定し、各パーティションが同じ州内の都市のデータを格納するように指定できます。これにより、特定の州や都市のクエリを高速化し、データの管理を容易にすることができます。

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

例2：時間の範囲と特定の州や都市に基づいてデータセンターの課金の詳細を頻繁にクエリするケースを考えます。テーブル作成時に、`dt`と`city`をパーティション列として指定します。このようにして、特定の日付と特定の州や都市のデータが同じパーティションに格納されるため、クエリ速度が向上し、データの管理が容易になります。

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

- リストパーディションでは、動的パーティショニングと一度に複数のパーティションを作成することができます。
- 現在、StarRocksの[共有データモード](../deployment/shared_data/s3.md)はこの機能をサポートしていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>;`ステートメントを使用してリストパーディショニングを使用して作成されたパーティションを削除すると、パーティション内のデータが直接削除され、復元することはできません。
- 現在、リストパーディショニングで作成されたパーティションを[バックアップおよびリストア](../administration/Backup_and_restore.md)することはできません。
- 現在、StarRocksはリストパーディション戦略で作成された基本テーブルの[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)の作成をサポートしていません。
```