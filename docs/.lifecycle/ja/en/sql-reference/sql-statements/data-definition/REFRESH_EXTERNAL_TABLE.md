---
displayed_sidebar: English
---

# 外部テーブルの更新

## 説明

StarRocks にキャッシュされた Hive および Hudi のメタデータを更新します。このステートメントは、以下のいずれかのシナリオで使用されます。

- **外部テーブル**: Hive 外部テーブルまたは Hudi 外部テーブルを使用して Apache Hive™ または Apache Hudi のデータをクエリする場合、このステートメントを実行して、StarRocks にキャッシュされた Hive テーブルまたは Hudi テーブルのメタデータを更新できます。
- **外部カタログ**: [Hive カタログ](../../../data_source/catalog/hive_catalog.md) または [Hudi カタログ](../../../data_source/catalog/hudi_catalog.md) を使用して Hive または Hudi のデータをクエリする場合、このステートメントを実行して、StarRocks にキャッシュされた Hive テーブルまたは Hudi テーブルのメタデータを更新できます。

## 基本概念

- **Hive 外部テーブル**: StarRocks に作成および格納されます。これを使用して Hive データをクエリできます。
- **Hudi 外部テーブル**: StarRocks に作成および格納されます。これを使用して Hudi データをクエリできます。
- **Hive テーブル**: Hive に作成され、格納されます。
- **Hudi テーブル**: Hudi に作成され、格納されます。

## 構文とパラメータ

以下では、さまざまなケースに基づく構文とパラメータについて説明します。

- 外部テーブル

    ```SQL
    REFRESH EXTERNAL TABLE table_name 
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**  | **必須** | **説明**                                              |
    | -------------- | ------------ | ------------------------------------------------------------ |
    | table_name     | はい          | Hive 外部テーブルまたは Hudi 外部テーブルの名前。    |
    | partition_name | いいえ           | Hive テーブルまたは Hudi テーブルのパーティションの名前。このパラメータを指定すると、StarRocks にキャッシュされている Hive テーブルと Hudi テーブルのパーティションのメタデータが更新されます。 |

- 外部カタログ

    ```SQL
    REFRESH EXTERNAL TABLE [external_catalog.][db_name.]table_name
    [PARTITION ('partition_name', ...)]
    ```

    | **パラメータ**    | **必須** | **説明**                                              |
    | ---------------- | ------------ | ------------------------------------------------------------ |
    | external_catalog | いいえ           | Hive カタログまたは Hudi カタログの名前。                  |
    | db_name          | いいえ           | Hive テーブルまたは Hudi テーブルが存在するデータベースの名前。 |
    | table_name       | はい          | Hive テーブルまたは Hudi テーブルの名前。                    |
    | partition_name   | いいえ           | Hive テーブルまたは Hudi テーブルのパーティションの名前。このパラメータを指定すると、StarRocks にキャッシュされている Hive テーブルと Hudi テーブルのパーティションのメタデータが更新されます。 |

## 使用上の注意

`ALTER_PRIV` 権限を持つユーザーのみがこのステートメントを実行して、StarRocks にキャッシュされた Hive テーブルと Hudi テーブルのメタデータを更新できます。

## 例

さまざまなケースでの使用例は以下の通りです。

### 外部テーブル

例 1: 外部テーブル `hive1` を指定して、StarRocks にキャッシュされた対応する Hive テーブルのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive1;
```

例 2: 外部テーブル `hudi1` と対応する Hudi テーブルのパーティションを指定して、StarRocks にキャッシュされた対応する Hudi テーブルのパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi1
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```

### 外部カタログ

例 1: `hive_table` の StarRocks にキャッシュされたメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hive_catalog.hive_db.hive_table;
```

または

```SQL
USE hive_catalog.hive_db;
REFRESH EXTERNAL TABLE hive_table;
```

例 2: `hudi_table` の StarRocks にキャッシュされたパーティションのメタデータを更新します。

```SQL
REFRESH EXTERNAL TABLE hudi_catalog.hudi_db.hudi_table
PARTITION ('date=2022-12-20', 'date=2022-12-21');
```
