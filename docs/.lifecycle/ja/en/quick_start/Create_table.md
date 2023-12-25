---
displayed_sidebar: English
---

# テーブルの作成

このクイックスタートチュートリアルでは、StarRocksでテーブルを作成するために必要な手順を説明し、StarRocksの基本的な機能をいくつか紹介します。

StarRocksインスタンスがデプロイされた後（詳細は[StarRocksのデプロイ](../quick_start/deploy_with_docker.md)を参照）、データベースとテーブルを作成して[データのロードとクエリ](../quick_start/Import_and_query.md)を行う必要があります。データベースとテーブルを作成するには、対応する[ユーザー権限](../administration/User_privilege.md)が必要です。このクイックスタートチュートリアルでは、StarRocksインスタンスで最高権限を持つデフォルトの`root`ユーザーを使用して、以下の手順を実行できます。

> **注記**
>
> このチュートリアルは、既存のStarRocksインスタンス、データベース、テーブル、およびユーザー権限を使用して完了できます。しかし、簡単にするために、チュートリアルで提供されるスキーマとデータの使用を推奨します。

## ステップ1: StarRocksにログイン

MySQLクライアントを使用してStarRocksにログインします。デフォルトユーザー`root`でログインでき、パスワードはデフォルトでは空です。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注記**
>
> - 別のFE MySQLサーバーポート（`query_port`、デフォルト：`9030`）を割り当てている場合は、`-P`の値をそれに応じて変更してください。
> - FE設定ファイルで`priority_networks`の設定項目を指定している場合は、`-h`の値をそれに応じて変更してください。

## ステップ2: データベースの作成

[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)を参照して、`sr_hub`という名前のデータベースを作成します。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQLを実行することで、このStarRocksインスタンス内のすべてのデータベースを表示できます。

## ステップ3: テーブルの作成

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

> **注記**
>
> - v3.1以降、テーブル作成時にDISTRIBUTED BY句でバケットキーを指定する必要はありません。StarRocksはランダムバケッティングをサポートしており、データをすべてのバケットにランダムに分散します。詳細は[ランダムバケッティング](../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。
> - デプロイしたStarRocksインスタンスにBEノードが1つしかないため、データレプリカの数を表すテーブルプロパティ`replication_num`を`1`に指定する必要があります。
> - テーブルタイプが指定されていない場合、デフォルトでDuplicate Keyテーブルが作成されます。[Duplicate Keyテーブル](../table_design/table_types/duplicate_key_table.md)を参照してください。
> - テーブルの列は、[データのロードとクエリ](../quick_start/Import_and_query.md)のチュートリアルでStarRocksにロードするデータのフィールドと正確に対応しています。
> - **本番環境での高性能を保証するために**、`PARTITION BY`句を使用してテーブルのデータパーティショニング計画を戦略的に立てることを強く推奨します。詳細は[パーティショニングとバケッティングのルール設計](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)を参照してください。

テーブルが作成されたら、DESC文を使用してテーブルの詳細を確認し、[SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md)を実行してデータベース内のすべてのテーブルを表示できます。StarRocksのテーブルはスキーマ変更をサポートしています。詳細は[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を参照してください。

## 次に行うこと

StarRocksテーブルの概念的な詳細については、[StarRocksテーブル設計](../table_design/StarRocks_table_design.md)を参照してください。

このチュートリアルで紹介した機能に加えて、StarRocksは以下もサポートしています：

- 様々な[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 複数の[テーブルタイプ](../table_design/table_types/table_types.md)
- 柔軟な[パーティショニング戦略](../table_design/Data_distribution.md#dynamic-partition-management)
- 従来のデータベースクエリインデックス、[ビットマップインデックス](../using_starrocks/Bitmap_index.md)や[ブルームフィルターインデックス](../using_starrocks/Bloomfilter_index.md)を含む
- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)
