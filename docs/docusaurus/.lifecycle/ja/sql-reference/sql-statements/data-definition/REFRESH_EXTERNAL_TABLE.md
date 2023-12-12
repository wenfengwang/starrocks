---
displayed_sidebar: "Japanese"
---

# 外部テーブルのリフレッシュ

## 説明

StarRocksにキャッシュされたHiveおよびHudiのメタデータを更新します。このステートメントは、次のシナリオの一つで使用されます:

- **外部テーブル**：Apache Hive™またはApache HudiでデータをクエリするためにHive外部テーブルまたはHudi外部テーブルを使用している場合、StarRocksにキャッシュされたHiveテーブルまたはHudiテーブルのメタデータを更新するためにこのステートメントを実行できます。
- **外部カタログ**：HiveまたはHudiでデータをクエリするために[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)または[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)を使用している場合、StarRocksにキャッシュされたHiveテーブルまたはHudiテーブルのメタデータを更新するためにこのステートメントを実行できます。

## 基本概念

- **Hive外部テーブル**：StarRocksに作成および保存されます。Hiveデータをクエリするために使用できます。
- **Hudi外部テーブル**：StarRocksに作成および保存されます。Hudiデータをクエリするために使用できます。
- **Hiveテーブル**：Hiveに作成および保存されます。
- **Hudiテーブル**：Hudiに作成および保存されます。

## 構文とパラメータ

異なるケースに基づいて、以下に構文とパラメータについて説明します。

- 外部テーブル

    ```SQL
    REFRESH EXTERNAL TABLE table_name 
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**  | **必須** | **説明**                                              |
    | -------------- | ------------ | ------------------------------------------------------------ |
    | table_name     | はい          | Hive外部テーブルまたはHudi外部テーブルの名前です。    |
    | partition_name | いいえ           | HiveテーブルまたはHudiテーブルのパーティションの名前です。このパラメータを指定すると、StarRocksにキャッシュされたHiveテーブルとHudiテーブルのパーティションのメタデータが更新されます。 |

- 外部カタログ

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]table_name
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**    | **必須** | **説明**                                              |
    | ---------------- | ------------ | ------------------------------------------------------------ |
    | external_catalog | いいえ           | HiveカタログまたはHudiカタログの名前です。                  |
    | db_name          | いいえ           | HiveテーブルまたはHudiテーブルが存在するデータベースの名前です。 |
    | table_name       | はい          | HiveテーブルまたはHudiテーブルの名前です。                    |
    | partition_name   | いいえ           | HiveテーブルまたはHudiテーブルのパーティションの名前です。このパラメータを指定すると、StarRocksにキャッシュされたHiveテーブルとHudiテーブルのパーティションのメタデータが更新されます。 |

## 使用上の注意

`ALTER_PRIV` 権限を持つユーザーのみが、HiveテーブルおよびHudiテーブルのメタデータをStarRocksにキャッシュするためにこのステートメントを実行できます。

## 例

異なるケースでの使用例は以下の通りです:

### 外部テーブル

例1: 外部テーブル `hive1` を指定して、StarRocksにキャッシュされた対応するHiveテーブルのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

例2: 外部テーブル `hudi1` および対応するHudiテーブルのパーティションを指定して、StarRocksにキャッシュされたパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部カタログ

例1: StarRocksにキャッシュされた `hive_table` のメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

または

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

例2: StarRocksにキャッシュされた `hudi_table` のパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```