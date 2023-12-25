---
displayed_sidebar: English
---

# リストパーティショニング

v3.1以降、StarRocksはリストパーティショニングをサポートしています。データは、各パーティションの事前定義された値リストに基づいてパーティション分割され、これによりクエリを高速化し、列挙値に基づいた管理を容易にします。

## 概要

各パーティションの列値リストを明示的に指定する必要があります。これらの値は、範囲パーティショニングで必要とされる連続する時間や数値の範囲とは異なり、連続している必要はありません。データのロード時、StarRocksはデータのパーティション列の値と各パーティションの事前定義された列値とのマッピングに基づいて、対応するパーティションにデータを格納します。

![list_partitioning](../assets/list_partitioning.png)

リストパーティショニングは、列に少数の列挙値が含まれるデータを格納するのに適しており、これらの列挙値に基づいてデータを頻繁にクエリし、管理する場合に有効です。例えば、列が地理的な場所、都道府県、カテゴリーを表す場合、列の各値は独立したカテゴリーを表します。列挙値に基づいてデータをパーティション分割することで、クエリのパフォーマンスを向上させ、データ管理を容易にすることができます。

**リストパーティショニングは、特にパーティション分割列ごとに複数の値を含むパーティションが必要なシナリオに有用です**。例えば、`City`列が個人の出身市を表し、都道府県や市区町村ごとにデータを頻繁にクエリし、管理する場合です。テーブル作成時に、`City`列をリストパーティショニングのためのパーティショニング列として使用し、同じ州内の様々な都市のデータを1つのパーティションに配置することを指定できます。例：`PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")`。この方法は、州や都市に基づくクエリを高速化し、データ管理を容易にします。

パーティションが各パーティション分割列の同じ値のデータのみを含む場合は、[式パーティショニング](./expression_partitioning.md)の使用を推奨します。

**リストパーティショニングと[式パーティショニング](./expression_partitioning.md)の比較**

リストパーティショニングと式パーティショニング（推奨）の主な違いは、リストパーティショニングではパーティションを一つずつ手動で作成する必要があることです。一方、式パーティショニングでは、ロード中にパーティションを自動的に作成し、パーティショニングを簡素化できます。また、ほとんどの場合、式パーティショニングはリストパーティショニングに取って代わることができます。以下の表にこの2つの具体的な比較を示します。

| パーティショニング方式                                      | **リストパーティショニング**                                        | **式パーティショニング**                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 構文                                                   | `PARTITION BY LIST (partition_columns) (    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                           |
| パーティション内のパーティション分割列ごとに複数の値 | サポートされています。パーティションは、パーティション分割列ごとに異なる値を持つデータを格納できます。例えば、ロードされたデータが`city`列に`Los Angeles`、`San Francisco`、`San Diego`の値を含む場合、すべてのデータは`pCalifornia`パーティションに格納されます。`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles", "San Francisco", "San Diego")    [, ...] )` | サポートされていません。パーティションは、パーティション分割列に同じ値を持つデータのみを格納します。例えば、式パーティショニングでは`PARTITION BY (city)`が使用されます。ロードされたデータが`city`列に`Los Angeles`、`San Francisco`、`San Diego`の値を含む場合、StarRocksは`pLosAngeles`、`pSanFrancisco`、`pSanDiego`の3つのパーティションを自動的に作成します。これらのパーティションは、それぞれ`city`列に`Los Angeles`、`San Francisco`、`San Diego`の値を持つデータを格納します。|

| データ読み込み前にパーティションを作成する                    | サポートされています。パーティションは、テーブル作成時に作成する必要があります。  | 必要ありません。パーティションは、データ読み込み中に自動的に作成されます。 |
| データ読み込み中にリストパーティションを自動的に作成する | サポートされていません。データ読み込み時に対応するパーティションが存在しない場合、エラーが返されます。 | サポートされています。データ読み込み時に対応するパーティションが存在しない場合、StarRocksはデータを格納するためにパーティションを自動的に作成します。各パーティションは、パーティション分割列の同じ値を持つデータのみを含むことができます。 |
| SHOW CREATE TABLE                                        | CREATE TABLE文でパーティション定義を返しました。 | データがロードされた後、ステートメントはCREATE TABLE文で使用されたパーティション句、つまり `PARTITION BY (partition_columns)` を含む結果を返します。ただし、返される結果には自動的に作成されたパーティションは表示されません。自動的に作成されたパーティションを表示するには、`SHOW PARTITIONS FROM <table_name>`を実行してください。 |

## 使用法

### 構文

```sql
PARTITION BY LIST (partition_columns) (
    PARTITION <partition_name> VALUES IN (value_list)
    [, ...]
)

partition_columns::= 
    <column> [,<column> [, ...] ]

value_list ::=
    value_item [, value_item [, ...] ]

value_item ::=
    { <value> | ( <value> [, <value> [, ...] ] ) }    
```

### パラメータ

| **パラメータ**      | **必須** | **説明**                                              |
| ------------------- | -------------- | ------------------------------------------------------------ |
| `partition_columns` | はい            | パーティション分割列の名前。パーティション分割列の値は、文字列（BINARYはサポートされていません）、日付または日時、整数、およびブール値を使用できます。パーティション分割列は`NULL`値を許容します。 |
| `partition_name`    | はい            | パーティション名。ビジネスシナリオに基づいて適切なパーティション名を設定し、異なるパーティション内のデータを区別することが推奨されます。 |
| `value_list`        | はい            | パーティション内のパーティション分割列の値のリストです。            |

### 例

例1: 州や市に基づいてデータセンターの請求詳細を頻繁にクエリする場合、テーブル作成時にパーティション分割列として`city`を指定し、各パーティションが同じ州内の都市のデータを格納するように指定することができます。この方法は、特定の州や市のクエリを高速化し、データ管理を容易にします。

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

例2: 時間範囲と特定の州や市に基づいてデータセンターの請求詳細を頻繁にクエリする場合、テーブル作成時にパーティション分割列として`dt`と`city`を指定することができます。これにより、特定の日付と特定の州や市のデータが同じパーティションに格納され、クエリの速度が向上し、データ管理が容易になります。

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

- リストパーティションは動的パーティショニングをサポートし、一度に複数のパーティションを作成することができます。
- 現在、StarRocksの[共有データモード](../deployment/shared_data/s3.md)はこの機能をサポートしていません。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` ステートメントを使用してリストパーティションを使用して作成されたパーティションを削除すると、そのパーティション内のデータは直接削除され、復旧することはできません。
- 現在、リストパーティショニングで作成されたパーティションの[バックアップと復元](../administration/Backup_and_restore.md)はできません。
- 現在、StarRocksはリストパーティショニング戦略で作成されたベーステーブルを用いた[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)の作成をサポートしていません。
