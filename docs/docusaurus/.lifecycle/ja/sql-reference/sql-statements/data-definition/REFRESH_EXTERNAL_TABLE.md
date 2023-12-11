---
displayed_sidebar: "Japanese"
---

# EXTERNAL TABLEのリフレッシュ

## 説明

StarRocksにキャッシュされているHiveおよびHudiのメタデータを更新します。このステートメントは次のいずれかのシナリオで使用されます：

- **外部テーブル**：Apache Hive™またはApache HudiでデータをクエリするためにHive外部テーブルまたはHudi外部テーブルを使用している場合、StarRocksにキャッシュされているHiveテーブルまたはHudiテーブルのメタデータを更新するためにこのステートメントを実行できます。
- **外部カタログ**：HiveまたはHudiでデータをクエリするために[Hiveカタログ](../../../data_source/catalog/hive_catalog.md)または[Hudiカタログ](../../../data_source/catalog/hudi_catalog.md)を使用している場合、StarRocksにキャッシュされているHiveテーブルまたはHudiテーブルのメタデータを更新するためにこのステートメントを実行できます。

## 基本的な概念

- **Hive外部テーブル**：StarRocksに作成および保存されます。これを使用してHiveデータをクエリできます。
- **Hudi外部テーブル**：StarRocksに作成および保存されます。これを使用してHudiデータをクエリできます。
- **Hiveテーブル**：Hiveに作成および保存されます。
- **Hudiテーブル**：Hudiに作成および保存されます。

## 構文とパラメータ

異なるケースに基づいて構文とパラメータについて説明します：

- 外部テーブル

    ```SQL
    REFRESH EXTERNAL TABLE テーブル名
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**  | **必須** | **説明**                                              |
    | -------------- | -------- | ----------------------------------------------------- |
    | テーブル名     | はい      | Hiveの外部テーブルまたはHudiの外部テーブルの名前。    |
    | partition_name | いいえ     | HiveテーブルまたはHudiテーブルのパーティションの名前。このパラメータを指定すると、StarRocksにキャッシュされているHiveテーブルおよびHudiテーブルのパーティションのメタデータが更新されます。 |

- 外部カタログ

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]テーブル名
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**    | **必須** | **説明**                                              |
    | ---------------- | -------- | ----------------------------------------------------- |
    | external_catalog | いいえ     | HiveカタログまたはHudiカタログの名前。                  |
    | db_name          | いいえ     | HiveテーブルまたはHudiテーブルが存在するデータベースの名前。 |
    | テーブル名       | はい      | HiveテーブルまたはHudiテーブルの名前。               |
    | partition_name   | いいえ     | HiveテーブルまたはHudiテーブルのパーティションの名前。このパラメータを指定すると、StarRocksにキャッシュされているHiveテーブルおよびHudiテーブルのパーティションのメタデータが更新されます。 |

## 使用上の注意

`ALTER_PRIV` 権限を持つユーザーのみが、StarRocksにキャッシュされているHiveテーブルおよびHudiテーブルのメタデータを更新するためにこのステートメントを実行できます。

## 例

異なるケースでの使用例は以下の通りです：

### 外部テーブル

例 1：外部テーブル `hive1` を指定してStarRocksにキャッシュされている対応するHiveテーブルのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

例 2：外部テーブル `hudi1` および対応するHudiテーブルのパーティションを指定してStarRocksにキャッシュされているパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部カタログ

例 1：StarRocksにキャッシュされている `hive_table` のメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

または

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

例 2：StarRocksにキャッシュされている `hudi_table` のパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```