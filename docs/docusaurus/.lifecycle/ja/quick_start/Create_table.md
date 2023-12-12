---
displayed_sidebar: "Japanese"
---

# テーブルを作成

このクイックスタートチュートリアルでは、StarRocksでテーブルを作成するための必要な手順を説明し、StarRocksの基本的な機能を紹介します。

StarRocksインスタンスが展開されたら（詳細は[StarRocksの展開](../quick_start/deploy_with_docker.md)を参照）、[データの読み込みとクエリ](../quick_start/Import_and_query.md)を行うためにデータベースとテーブルを作成する必要があります。データベースとテーブルを作成するには対応する[ユーザ権限](../administration/User_privilege.md)が必要です。このクイックスタートチュートリアルでは、StarRocksインスタンスで最高の権限を持つデフォルトの`root`ユーザを使用して以下の手順を実行できます。

> **注意**
>
> 既存のStarRocksインスタンス、データベース、テーブル、およびユーザ権限を使用してこのチュートリアルを完了することができますが、シンプルにするために、チュートリアルが提供するスキーマとデータを使用することをお勧めします。

## ステップ1：StarRocksにログイン

MySQLクライアントを使用してStarRocksにログインします。デフォルトのユーザ`root`でログインし、デフォルトではパスワードは空です。

```Plain
mysql -h <fe_ip> -P<fe_query_port> -uroot
```

> **注意**
>
> - 異なるFE MySQLサーバーポート（`query_port`、デフォルト：`9030`）が割り当てられている場合は、`-P`の値を適宜変更してください。
> - FE構成ファイルで構成項目`priority_networks`を指定した場合は、`-h`の値を適切に変更してください。

## ステップ2：データベースを作成

[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)を参照して、`sr_hub`という名前のデータベースを作成します。

```SQL
CREATE DATABASE IF NOT EXISTS sr_hub;
```

このStarRocksインスタンスで利用可能なすべてのデータベースを[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) SQLを実行して確認できます。

## ステップ3：テーブルを作成

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
> - v3.1以降、テーブルを作成する際にDISTRIBUTED BY句でバケット化キーを指定する必要はありません。StarRocksはランダムバケット化をサポートしており、データをすべてのバケットにランダムに分散させます。詳細については、[ランダムバケット化](../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。
> - デプロイしたStarRocksインスタンスにはBEノードが1つしかないため、データレプリカの数を表すテーブルプロパティ`replication_num`を`1`で指定する必要があります。
> - [テーブルタイプ](../table_design/table_types/table_types.md)が指定されていない場合、デフォルトでDuplicate Keyテーブルが作成されます。[Duplicate Keyテーブル](../table_design/table_types/duplicate_key_table.md)を参照してください。
> - テーブルの列は、チュートリアルでStarRocksにロードするデータのフィールドに正確に対応しています。
> - **本番環境**での高いパフォーマンスを確保するために、`PARTITION BY`句を使用してテーブルのデータパーティションプランを戦略的に立案することを強くお勧めします。詳細については、[パーティショニングおよびバケット化ルールの設計](../table_design/Data_distribution.md#design-partitioning-and-bucketing-rules)を参照してください。

テーブルが作成されたら、DESCステートメントを使用してテーブルの詳細を確認し、[SHOW TABLES](../sql-reference/sql-statements/data-manipulation/SHOW_TABLES.md)を実行することでデータベース内のすべてのテーブルを表示できます。StarRocksのテーブルではスキーマの変更がサポートされています。詳細については、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を参照してください。

## 次に何をするか

StarRocksテーブルの概念的な詳細については、[StarRocks テーブルデザイン](../table_design/StarRocks_table_design.md)を参照してください。

このチュートリアルでデモンストレーションされている機能に加えて、StarRocksは以下もサポートしています。

- 様々な[データ型](../sql-reference/sql-statements/data-types/BIGINT.md)
- 複数の[テーブルタイプ](../table_design/table_types/table_types.md)
- 柔軟な[パーティショニング戦略](../table_design/Data_distribution.md#dynamic-partition-management)
- [ビットマップインデックス](../using_starrocks/Bitmap_index.md)や[ブルームフィルターインデックス](../using_starrocks/Bloomfilter_index.md)を含むクラシックなデータベースクエリインデックス
- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)
