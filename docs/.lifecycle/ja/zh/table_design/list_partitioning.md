---
displayed_sidebar: Chinese
---

# List 分区

自 v3.1 以降、StarRocks は List 分区をサポートしており、データは明示的に定義された列挙値リストに従って分割されます。これは、列挙値に基づいてデータをクエリおよび管理するのに適しています。

## 機能紹介

List 分区に含まれる列挙値リストを明示的にリストアップする必要があり、値は連続している必要はありません。これは、連続した日付や数値範囲を含む Range 分区とは異なります。新しいデータがテーブルにインポートされると、StarRocks はデータの分区列の値と分区のマッピング関係に基づいて、データを適切な分区に割り当てます。

![list_partitioning](../assets/list_partitioning.png)

List 分区は、少数の列挙値列を持つデータを格納し、列の列挙値に基づいてデータを頻繁にクエリおよび管理するシナリオに適しています。例えば、地理的位置、状態、カテゴリーを表す列などです。各列の値は独立したカテゴリーを表します。列の列挙値に基づいてデータを分割することで、クエリのパフォーマンスを向上させ、データ管理を容易にします。

**特に、1つの分区に複数の分区列の値を含める必要があるシナリオに適しています**。例えば、テーブルに `City` 列があり、個々の都市に属していることを示し、州や都市に基づいてデータを頻繁にクエリおよび管理する場合、テーブル作成時に `City` 列を分区列として使用し、List 分区を行い、同じ州に属する複数の都市のデータを同じ分区に割り当てることができます。`PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")` これにより、クエリが加速され、データ管理が容易になります。

1つの分区に分区列の1つの値のみを含める場合は、[式分区](./expression_partitioning.md)の使用をお勧めします。

**List 分区と[式分区](./expression_partitioning.md)の違い**

両者の主な違いは、List 分区では分区を手動で1つずつ作成する必要があることです。一方、式分区（推奨）は、インポート時に自動的に分区を作成し、分区の作成を簡素化し、ほとんどの場合で List 分区を置き換えることができます。具体的な比較は以下の表の通りです：

| 分区方式                                     | **List 分区**                                                | **式分区**                                               |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 文法                                         | `PARTITION BY LIST (partition_columns)（    PARTITION <partition_name> VALUES IN (value_list)    [, ...] )` | `PARTITION BY <partition_columns>`                           |
| 1つの分区に複数の分区列の値を含む                 | 対応。1つの分区に複数の分区列の値を含むことができます。例えば、インポートされたデータの `city` 列の値が `Los Angeles`、`San Francisco`、または `San Diego` の場合、それらはすべて `pCalifornia` 分区に割り当てられます。<br />`PARTITION BY LIST (city) (    PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego")    [, ...] )` | 対応していません。1つの分区には、分区列に適用された式を計算して得られた1つの値のみを含みます。例えば、式分区 `PARTITION BY (city)` を使用し、インポートされた複数行のデータの `city` 列の値に `Los Angeles`、`San Francisco`、または `San Diego` が含まれている場合、`pLosAngeles`、`pSanFrancisco`、`pSanDiego` の3つの分区が自動的に作成され、それぞれ `city` 列の値が `Los Angeles`、`San Francisco`、`San Diego` のデータを含みます。|
| データインポート前に分区を事前に作成する必要がある                     | 必須。テーブル作成時に分区を作成する必要があります。                             | 不要。データインポート時に自動的に分区が作成されます。                             |
| データインポート時に自動的に分区を作成する                         | 対応していません。データインポート時に、テーブルにデータに対応する分区が存在しない場合、StarRocks は自動的に分区を作成せず、エラーが発生します。 | 対応。データインポート時に、テーブルにデータに対応する分区が存在しない場合、StarRocks は自動的に分区を作成し、データをインポートします。そして、1つの分区には分区列の1つの値のみを含みます。 |
| SHOW CREATE TABLE                            | テーブル作成文で定義された分区を返します。                                     | データインポート後にこのコマンドを実行すると、結果にはテーブル作成時の分区節、つまり `PARTITION BY (partition_columns)` が含まれますが、自動的に作成された分区は返されません。自動的に作成された分区を表示するには、`SHOW PARTITIONS FROM table_name;` を実行してください。 |

## 使用方法

### 文法

```bnf
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

### パラメータ説明

| パラメータ                | 必須 | 説明                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| `partition_columns` | はい       | 分区列。<br />分区列の値は文字列（BINARY 除く）、日付（DATE および DATETIME）、整数、およびブール値をサポートします。分区列の値は `NULL` をサポートしません。 |
| `partition_name`    | はい       | 分区名。ビジネスシナリオに応じて合理的な分区名を設定することをお勧めします。これにより、異なる分区に含まれるデータのカテゴリを区別しやすくなります。 |
| `value_list`        |          | 分区内の分区列の列挙値リスト。                                   |

## 例

例1：州や都市に基づいてデータセンターの料金明細を頻繁にクエリする場合、テーブル作成時に分区列を都市 `city` と指定し、各分区に含まれる都市が同じ州に属するように指定することで、特定の州や都市のデータを迅速にクエリし、特定の州や都市に基づいてデータを管理することが容易になります。

```SQL
CREATE TABLE t_recharge_detail2 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY LIST (city) (
   PARTITION pCalifornia VALUES IN ("Los Angeles","San Francisco","San Diego"), -- これらの都市は同じ州に属しています
   PARTITION pTexas VALUES IN ("Houston","Dallas","Austin")
)
DISTRIBUTED BY HASH(`id`);
```

例2：特定の日付範囲と特定の州または都市に基づいてデータセンターの料金明細を頻繁にクエリする場合、テーブル作成時に分区列を日付 `dt` と都市 `city` と指定します。これにより、特定の日付と特定の州または都市に属するデータが同じ分区にグループ化され、クエリが迅速化され、データ管理が容易になります。

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

## 使用制限

- [動的分区](./dynamic_partitioning.md)および[List 分区の一括作成](./Data_distribution.md#range-分区)はサポートされていません。
- StarRocks の[計算とストレージの分離モード](../deployment/shared_data/s3.md)はバージョン 3.1.1 からこの機能をサポートしています。
- `ALTER TABLE <table_name> DROP PARTITION <partition_name>;` を使用すると、分区は直接削除され、復元することはできません。
- List 分区は現在、[バックアップと復元](../administration/Backup_and_restore.md)をサポートしていません。
- [非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)は、List 分区を使用するベーステーブルに基づいて作成することはできません。
