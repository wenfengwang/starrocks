---
displayed_sidebar: Chinese
---
# テーブルの作成

このドキュメントでは、StarRocksでテーブルを作成し、関連する操作を行う方法について説明します。

> **注意**
>
> デフォルトカタログの下に [CREATE DATABASE](../administration/privilege_item.md) の権限を持つユーザーのみがデータベースを作成できます。データベース内でテーブルを作成するためには、そのデータベースで CREATE TABLE の権限を持つユーザーである必要があります。

## StarRocksに接続する

[StarRocksクラスタをデプロイ](../quick_start/deploy_with_docker.md)した後、MySQLクライアントを使用してStarRocksに接続することができます。MySQLクライアントを使用して、任意のFEノードの`query_port`（デフォルトは`9030`）に接続します。StarRocksにはデフォルトで`root`ユーザーが組み込まれており、パスワードはデフォルトで空です。

```shell
mysql -h <fe_host> -P9030 -u root
```

## データベースの作成

`example_db`データベースを作成します。

> **注意**
>
> データベース名、テーブル名、列名などの変数を指定する場合、予約語を使用する場合はバッククォート（`）で囲む必要があります。StarRocksの予約語のリストについては、[キーワード](../sql-reference/sql-statements/keywords.md#保留关键字)を参照してください。

```sql
CREATE DATABASE example_db;
```

`SHOW DATABASES;`コマンドを使用して、現在のStarRocksクラスタに存在するすべてのデータベースを表示できます。

```Plain Text
MySQL [(none)]> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| _statistics_       |
| example_db         |
| information_schema |
+--------------------+
3 rows in set (0.00 sec)
```

> 注意：MySQLのテーブル構造と同様に、`information_schema`には現在のStarRocksクラスタのメタデータ情報が含まれていますが、一部の統計情報はまだ完全ではありません。データベースのメタデータ情報を取得するには、`DESC table_name`などのコマンドを使用することをお勧めします。

## テーブルの作成

作成したデータベース内にテーブルを作成します。

StarRocksは、さまざまなデータモデルをサポートしており、さまざまなアプリケーションシナリオに適用することができます。以下の例は、[詳細テーブルモデル](../table_design/table_types/duplicate_key_table.md)を基にした作成文です。

詳細な作成文法については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

```sql
use example_db;
CREATE TABLE IF NOT EXISTS `detailDemo` (
    `recruit_date`  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    `region_num`    TINYINT        COMMENT "range [-128, 127]",
    `num_plate`     SMALLINT       COMMENT "range [-32768, 32767] ",
    `tel`           INT            COMMENT "range [-2147483648, 2147483647]",
    `id`            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
    `password`      LARGEINT       COMMENT "range [-2^127 + 1 ~ 2^127 - 1]",
    `name`          CHAR(20)       NOT NULL COMMENT "range char(m),m in (1-255)",
    `profile`       VARCHAR(500)   NOT NULL COMMENT "upper limit value 1048576 bytes",
    `hobby`         STRING         NOT NULL COMMENT "upper limit value 65533 bytes",
    `leave_time`    DATETIME       COMMENT "YYYY-MM-DD HH:MM:SS",
    `channel`       FLOAT          COMMENT "4 bytes",
    `income`        DOUBLE         COMMENT "8 bytes",
    `account`       DECIMAL(12,4)  COMMENT "",
    `ispass`        BOOLEAN        COMMENT "true/false"
) ENGINE=OLAP
DUPLICATE KEY(`recruit_date`, `region_num`)
PARTITION BY RANGE(`recruit_date`)
(
    PARTITION p20220311 VALUES [('2022-03-11'), ('2022-03-12')),
    PARTITION p20220312 VALUES [('2022-03-12'), ('2022-03-13')),
    PARTITION p20220313 VALUES [('2022-03-13'), ('2022-03-14')),
    PARTITION p20220314 VALUES [('2022-03-14'), ('2022-03-15')),
    PARTITION p20220315 VALUES [('2022-03-15'), ('2022-03-16'))
)
DISTRIBUTED BY HASH(`recruit_date`, `region_num`);
```

> 注意
>
> * StarRocksでは、フィールド名は大文字と小文字を区別せず、テーブル名は大文字と小文字を区別します。
> * 3.1以降、テーブルを作成する際に分桶キー（DISTRIBUTED BY句）を設定する必要はありません。StarRocksはデフォルトでランダムな分桶を使用し、データをパーティションのすべての分桶にランダムに分散させます。詳細については、[ランダム分桶](../table_design/Data_distribution.md#随机分桶自-v31)を参照してください。

### テーブル作成文の説明

#### ソートキー

StarRocksは、テーブル内のデータを指定された列でソートして格納します。これらの列はソートキーと呼ばれます。詳細モデルでは、`DUPLICATE KEY`でソートキーが指定されます。上記の例では、`recruit_date`と`region_num`の2つの列がソートキーです。

> 注意：ソートキーは、他の列よりも前に定義する必要があります。ソートキーの詳細な説明と、異なるデータモデルのテーブルの設定方法については、[ソートキー](../table_design/Sort_key.md)を参照してください。

#### フィールドタイプ

StarRocksテーブルでは、上記の例で列挙されているフィールドタイプ以外にも、さまざまなフィールドタイプをサポートしています。[BITMAPタイプ](../using_starrocks/Using_bitmap.md)、[HLLタイプ](../using_starrocks/Using_HLL.md)、[ARRAYタイプ](../sql-reference/sql-statements/data-types/Array.md)などのフィールドタイプの詳細については、[データ型のセクション](../sql-reference/sql-statements/data-types/BIGINT.md)を参照してください。

> 注意：テーブルを作成する際は、できるだけ正確なタイプを使用するようにしてください。たとえば、整数データには文字列型を使用しないでください。INT型で十分なデータにBIGINT型を使用しないでください。正確なデータ型を使用することで、データベースのパフォーマンスを最大限に引き出すことができます。

#### パーティションと分桶

`PARTITION`キーワードは、テーブルに[パーティション](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#partition_desc)を作成するために使用されます。上記の例では、`recruit_date`を使用して範囲パーティションを作成し、11日から15日までの各日に1つのパーティションを作成しています。StarRocksは動的パーティションをサポートしています。詳細については、[動的パーティション管理](../table_design/dynamic_partitioning.md)を参照してください。**生産環境のクエリパフォーマンスを最適化するために、テーブルに適切なデータパーティションプランを指定することを強くお勧めします。**

`DISTRIBUTED`キーワードは、テーブルに[分桶](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#distribution_desc)を作成するために使用されます。上記の例では、`recruit_date`と`region_num`の2つのフィールドを分桶列として使用しています。また、StarRocksは2.5.7以降、分桶数を自動的に設定する機能をサポートしていますので、分桶数を手動で設定する必要はありません。詳細については、[分桶数の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

適切なパーティションと分桶の設計により、テーブルのクエリパフォーマンスを最適化することができます。パーティションと分桶の列の選択方法については、[データ分布](../table_design/Data_distribution.md)を参照してください。

#### データモデル

`DUPLICATE`キーワードは、現在のテーブルが詳細モデルであることを示し、`KEY`内のリストは現在のテーブルのソートキーを示します。StarRocksは、[詳細モデル](../table_design/table_types/duplicate_key_table.md)、[集計モデル](../table_design/table_types/aggregate_table.md)、[更新モデル](../table_design/table_types/unique_key_table.md)、[主キーモデル](../table_design/table_types/primary_key_table.md)など、さまざまなデータモデルをサポートしています。さまざまなモデルはさまざまなビジネスシナリオに適用され、適切な選択によりクエリの効率を最適化することができます。

#### インデックス

StarRocksは、デフォルトでキーカラムにスパースインデックスを作成してクエリを高速化します。詳細なルールについては、[ソートキー](../table_design/Sort_key.md)を参照してください。サポートされるインデックスの種類には、[ビットマップインデックス](../using_starrocks/Bitmap_index.md)、[ブルームフィルタインデックス](../using_starrocks/Bloomfilter_index.md)などがあります。

> 注意：インデックスの作成には、テーブルモデルと列に要件があります。詳細については、対応するインデックスの紹介セクションを参照してください。

#### ENGINEタイプ

デフォルトのENGINEタイプは`olap`であり、StarRocksクラスタ内部のテーブルに対応しています。その他のオプションには、`mysql`、`elasticsearch`、`hive`、`jdbc`（2.3以降）、`hudi`（2.2以降）、`iceberg`があり、それぞれ作成されるテーブルは[外部テーブル](../data_source/External_table.md)です。

## テーブル情報の表示

SQLコマンドを使用して、テーブルの関連情報を表示することができます。

* 現在のデータベース内のすべてのテーブルを表示する

```sql
SHOW TABLES;
```

* テーブルの構造を表示する

```sql
DESC table_name;
```

例：

```sql
DESC detailDemo;
```

* テーブルの作成文を表示する

```sql
SHOW CREATE TABLE table_name;
```

例：

```sql
SHOW CREATE TABLE detailDemo;
```

<br/>

## テーブル構造の変更

StarRocksは、さまざまなDDL操作をサポートしています。

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#schema-change)コマンドを使用して、テーブルのスキーマを変更することができます。列の追加、列の削除、列の型の変更（列名の変更は現在サポートされていません）、列の順序の変更などが可能です。

### 列の追加

例えば、上記で作成したテーブルに、`ispass`列の後に`uv`列を追加し、BIGINT型でデフォルト値を`0`に設定します。

```sql
ALTER TABLE detailDemo ADD COLUMN uv BIGINT DEFAULT '0' after ispass;
```

### 列の削除

上記の手順で追加した列を削除します。

> 注意
>
> 上記の手順で`uv`を追加した場合は、後続のクイックスタートの内容を実行できるように、この列を削除してください。

```sql
ALTER TABLE detailDemo DROP COLUMN uv;
```

### テーブル構造変更ジョブのステータスの確認

テーブルの構造変更は非同期操作です。正常に送信された後、ジョブのステータスを確認するには、次のコマンドを使用できます。

```sql
SHOW ALTER TABLE COLUMN\G;
```

ジョブのステータスがFINISHEDの場合、ジョブが完了し、新しいテーブルの構造変更が有効になります。

スキーマの変更が完了した後は、次のコマンドを使用して最新のテーブル構造を表示できます。

```sql
DESC table_name;
```

例：

```Plain Text
MySQL [example_db]> desc detailDemo;

+--------------+-----------------+------+-------+---------+-------+
| Field        | Type            | Null | Key   | Default | Extra |
+--------------+-----------------+------+-------+---------+-------+
| recruit_date | DATE            | No   | true  | NULL    |       |
| region_num   | TINYINT         | Yes  | true  | NULL    |       |
| num_plate    | SMALLINT        | Yes  | false | NULL    |       |
| tel          | INT             | Yes  | false | NULL    |       |
| id           | BIGINT          | Yes  | false | NULL    |       |
| password     | LARGEINT        | Yes  | false | NULL    |       |
| name         | CHAR(20)        | No   | false | NULL    |       |
| profile      | VARCHAR(500)    | No   | false | NULL    |       |
| hobby        | VARCHAR(65533)  | No   | false | NULL    |       |
| leave_time   | DATETIME        | Yes  | false | NULL    |       |
| channel      | FLOAT           | Yes  | false | NULL    |       |
| income       | DOUBLE          | Yes  | false | NULL    |       |
| account      | DECIMAL64(12,4) | Yes  | false | NULL    |       |
| ispass       | BOOLEAN         | Yes  | false | NULL    |       |
| uv           | BIGINT          | Yes  | false | 0       |       |
+--------------+-----------------+------+-------+---------+-------+
15 rows in set (0.00 sec)
```

### テーブル構造変更のキャンセル

現在実行中のジョブをキャンセルするには、次のコマンドを使用できます。

```sql
CANCEL ALTER TABLE COLUMN FROM table_name\G;
```

## ユーザーの作成と権限の付与

`example_db`データベースが作成された後、`test`ユーザーを作成し、`example_db`の読み書き権限を付与することができます。

```sql
CREATE USER 'test' IDENTIFIED by '123456';
GRANT ALL on example_db.* to test;
```

付与された`test`アカウントでログインすると、`example_db`データベースを操作することができます。

```bash
mysql -h 127.0.0.1 -P9030 -utest -p123456
```

<br/>

## 次のステップ

テーブルが作成されたら、[データのインポートとクエリ](../quick_start/Import_and_query.md)を行うことができます。