---
displayed_sidebar: Chinese
---

# 外部テーブルの更新

## 機能

このステートメントは、StarRocksにキャッシュされたApache Hive™およびApache Hudiのメタデータを更新するために使用され、主に以下の二つの使用シナリオがあります：

- **外部テーブル**: Hive外部テーブルおよびHudi外部テーブルを使用してHiveおよびHudiのデータをクエリする際に、キャッシュされたHiveおよびHudiのメタデータを更新するためにこのステートメントを使用できます。
- **External catalog**: [Hive catalog](../../../data_source/catalog/hive_catalog.md)および[Hudi catalog](../../../data_source/catalog/hudi_catalog.md)を使用してHiveおよびHudiのデータをクエリする際に、キャッシュされたHiveおよびHudiのメタデータを更新するためにこのステートメントを使用できます。

> **注意**
>
> 対応する外部テーブルのALTER権限を持つユーザーのみがこの操作を実行できます。

## 基本概念

- **Hive外部テーブル**: StarRocksに作成され保存されたテーブルで、Hiveクラスターのデータをクエリするために使用されます。
- **Hudi外部テーブル**: StarRocksに作成され保存されたテーブルで、Hudiクラスターのデータをクエリするために使用されます。
- **Hiveテーブル**: Hiveに作成され保存されたテーブル。
- **Hudiテーブル**: Hudiに作成され保存されたテーブル。

## 语法とパラメータ説明

異なる使用シナリオに応じた、対応する構文とパラメータの説明は以下の通りです：

- 外部テーブル

    ```SQL
    REFRESH EXTERNAL TABLE <table_name>
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**   | **必須** | **説明**                                                     |
    | -------------- | -------- | ------------------------------------------------------------ |
    | table_name     | はい     | Hive外部テーブルまたはHudi外部テーブルの名前。                                |
    | partition_name | いいえ   | HiveテーブルまたはHudiテーブルのパーティション名。指定された場合、指定されたパーティションのキャッシュされたHiveテーブルまたはHudiテーブルのメタデータを更新します。 |

- External catalog

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]<table_name>
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**     | **必須** | **説明**                                                     |
    | ---------------- | -------- | ------------------------------------------------------------ |
    | external_catalog | いいえ   | Hive catalogまたはHudi catalogの名前。                          |
    | db_name          | いいえ   | HiveテーブルまたはHudiテーブルが存在するデータベース名。                            |
    | table_name       | はい     | HiveテーブルまたはHudiテーブルの名前。                                        |
    | partition_name   | いいえ   | HiveテーブルまたはHudiテーブルのパーティション名。指定された場合、指定されたパーティションのキャッシュされたHiveテーブルまたはHudiテーブルのメタデータを更新します。 |

## 注意事項

外部テーブルのALTER権限を持つユーザーのみが、キャッシュされたメタデータを更新するためにこのステートメントを実行できます。

## 例

異なる使用シナリオに応じた、対応する例は以下の通りです。

### 外部テーブル

例1：外部テーブル`hive1`に対応するHiveテーブルのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

例2：外部テーブル`hudi1`に対応するHudiテーブルの`p1`および`p2`パーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('p1', 'p2');
```

### External catalog

例1：キャッシュされたHiveテーブル`hive_table`のメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

または

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

例2：キャッシュされたHudiテーブル`hudi_table`の`p1`および`p2`パーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('p1', 'p2');
```
