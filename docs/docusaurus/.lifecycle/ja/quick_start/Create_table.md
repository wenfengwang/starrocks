---
displayed_sidebar: "Japanese"
---

# テーブルを作成する

このクイックスタートチュートリアルでは、StarRocksでテーブルを作成するための必要な手順を説明し、StarRocksの基本的な機能を紹介します。

StarRocksインスタンスをデプロイした後（詳細は[StarRocksをデプロイ](../quick_start/deploy_with_docker.md)を参照）、[データのロードとクエリ](../quick_start/Import_and_query.md)を行うために、データベースとテーブルを作成する必要があります。データベースとテーブルの作成には、対応する[user privilege](../administration/User_privilege.md)が必要です。このクイックスタートチュートリアルでは、StarRocksインスタンスに最も高い権限を持つデフォルトの `root` ユーザーで次の手順を実行できます。

> **注**
>
> 既存のStarRocksインスタンス、データベース、テーブル、およびユーザー権限を使用してこのチュートリアルを完了することができます。ただし、簡単のために、チュートリアルが提供するスキーマとデータを使用することをお勧めします。

## ステップ1：StarRocksにログインする

MySQLクライアントを使用してStarRocksにログインします。デフォルトのユーザー `root` でログインし、デフォルトではパスワードは空です。

```プレーン
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注**
>
> - 別のFE MySQLサーバーポート（`query_port`、デフォルト：`9030`）が割り当てられている場合は、`-P`の値を適宜変更してください。
> - FE構成ファイルで構成項目`priority_networks`を指定した場合は、`-h`の値を適切に変更してください。

## ステップ2：データベースを作成する

[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)を参照して、`sr_hub`という名前のデータベースを作成します。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQLを実行して、このStarRocksインスタンスのすべてのデータベースを表示できます。

## ステップ3：テーブルを作成する

`USE sr_hub`を実行して`sr_hub`データベースに切り替え、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照して`sr_member`という名前のテーブルを作成します。

```SQL
USE sr_hub;
CREATE TABLE IF NOT EXISTS sr_member (
    sr_id            INT,
    name             STRING,
    city_code        INT,
    reg_date         DATE,
    verified         BOOLEAN
)
PARTITION BY RANGE(reg_date)
(
    PARTITION p1 VALUES [('2022-03-13'), ('2022-03-14')),
    PARTITION p2 VALUES [('2022-03-14'), ('2022-03-15')),
    PARTITION p3 VALUES [('2022-03-15'), ('2022-03-16')),
    PARTITION p4 VALUES [('2022-03-16'), ('2022-03-17')),
    PARTITION p5 VALUES [('2022-03-17'), ('2022-03-18'))
)
DISTRIBUTED BY HASH(city_code);
```

> **注**
>
> - v3.1以降、テーブルの作成時にDISTRIBUTED BY句でバケット化キーを指定する必要はありません。StarRocksはランダムバケティングをサポートしており、データをランダムにすべてのバケットに分散させます。詳細については、[ランダムバケティング](../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。
> - デプロイしたStarRocksインスタンスにBEノードが1つしかないため、データレプリカの数を表すテーブルプロパティ`replication_num`は`1`として指定する必要があります。
> - [table type](../table_design/table_types/table_types.md)が指定されていない場合、デフォルトでDuplicate Keyテーブルが作成されます。[Duplicate Key table](../table_design/table_types/duplicate_key_table.md)を参照してください。
> - テーブルの列は、チュートリアルでStarRocksにロードするデータのフィールドに厳密に対応しています。
> - 本番環境で高いパフォーマンスを確保するためには、`PARTITION BY`句を使用してテーブルのデータパーティショニング計画を計画することを強くお勧めします。詳細については、[Design partitioning and bucketing rules](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)を参照してください。

テーブルが作成されたら、DESCステートメントを使用してテーブルの詳細を確認し、[SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md)を実行してデータベース内のすべてのテーブルを表示できます。StarRocksのテーブルはスキーマの変更をサポートしています。詳細については、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を参照してください。

## 次に何をするか

StarRocksテーブルの概念的な詳細について詳しく知るには、[StarRocks Table Design](../table_design/StarRocks_table_design.md)を参照してください。

このチュートリアルでデモンストレーションされた機能に加えて、StarRocksは次のような機能もサポートしています：

- 様々な[data types](../sql-reference/sql-statements/data-types/BIGINT.md)
- 複数の[table types](../table_design/table_types/table_types.md)
- 柔軟な[partitioning strategies](../table_design/Data_distribution.md#dynamic-partition-management)
- [bitmap index](../using_starrocks/Bitmap_index.md)および[bloom filter index](../using_starrocks/Bloomfilter_index.md)を含むクラシックなデータベースクエリインデックス
- [Materialized view](../using_starrocks/Materialized_view.md)