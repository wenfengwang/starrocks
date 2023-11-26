---
displayed_sidebar: "Japanese"
---

# テーブルの作成

このクイックスタートチュートリアルでは、StarRocksでテーブルを作成するための必要な手順と、StarRocksの基本的な機能を紹介します。

StarRocksインスタンスが展開された後（詳細は[StarRocksの展開](../quick_start/deploy_with_docker.md)を参照）、データを[ロードおよびクエリ](../quick_start/Import_and_query.md)するためにデータベースとテーブルを作成する必要があります。データベースとテーブルの作成には、対応する[ユーザ権限](../administration/User_privilege.md)が必要です。このクイックスタートチュートリアルでは、StarRocksインスタンス上で最も高い権限を持つデフォルトの`root`ユーザで以下の手順を実行できます。

> **注意**
>
> 既存のStarRocksインスタンス、データベース、テーブル、およびユーザ権限を使用してこのチュートリアルを完了することもできます。ただし、簡単のために、チュートリアルが提供するスキーマとデータを使用することをお勧めします。

## ステップ1：StarRocksにログインする

MySQLクライアントを使用してStarRocksにログインします。デフォルトのユーザ`root`でログインし、パスワードはデフォルトで空です。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
>
> - 異なるFE MySQLサーバポート（`query_port`、デフォルト：`9030`）を割り当てた場合は、`-P`の値を適宜変更してください。
> - FE設定ファイルで構成項目`priority_networks`を指定した場合は、`-h`の値を適宜変更してください。

## ステップ2：データベースの作成

[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)を参照して、`sr_hub`という名前のデータベースを作成します。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

このStarRocksインスタンスのすべてのデータベースを表示するには、[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQLを実行します。

## ステップ3：テーブルの作成

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

> **注意**
>
> - v3.1以降、テーブルを作成する際にDISTRIBUTED BY句でバケットキーを指定する必要はありません。StarRocksはランダムバケットをサポートしており、データをすべてのバケットにランダムに分散させます。詳細については、[ランダムバケット](../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。
> - デプロイしたStarRocksインスタンスには1つのBEノードしかないため、データのレプリカ数を表すテーブルプロパティ`replication_num`を`1`として指定する必要があります。
> - [テーブルタイプ](../table_design/table_types/table_types.md)が指定されていない場合、デフォルトでDuplicate Keyテーブルが作成されます。詳細については、[Duplicate Keyテーブル](../table_design/table_types/duplicate_key_table.md)を参照してください。
> - テーブルの列は、[データのロードとクエリ](../quick_start/Import_and_query.md)のチュートリアルでStarRocksにロードするデータのフィールドとまったく対応しています。
> - 本番環境での高いパフォーマンスを保証するためには、`PARTITION BY`句を使用してテーブルのデータパーティショニング計画を戦略的に立てることを強くお勧めします。詳細な手順については、[パーティショニングとバケット化のルールの設計](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)を参照してください。

テーブルが作成された後、DESCステートメントを使用してテーブルの詳細を確認し、[SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md)を実行してデータベース内のすべてのテーブルを表示できます。StarRocksのテーブルはスキーマの変更をサポートしています。詳細については、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を参照してください。

## 次に何をするか

StarRocksテーブルの概念的な詳細については、[StarRocksテーブル設計](../table_design/StarRocks_table_design.md)を参照してください。

このチュートリアルで示した機能に加えて、StarRocksは以下の機能もサポートしています。

- 様々な[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 複数の[テーブルタイプ](../table_design/table_types/table_types.md)
- 柔軟な[パーティショニング戦略](../table_design/Data_distribution.md#dynamic-partition-management)
- ビットマップインデックスやブルームフィルタインデックスなどのクラシックなデータベースクエリインデックス
- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)
