---
displayed_sidebar: "Japanese"
---

# 使用方法

## 説明

セッションのアクティブなデータベースを指定します。その後、テーブルの作成やクエリの実行などの操作を行うことができます。

## 構文

```SQL
USE [<catalog_name>.]<db_name>
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | --------- | ------------------------------------------------------------- |
| catalog_name   | いいえ    | カタログ名。<ul><li>このパラメータが指定されていない場合は、デフォルトで `default_catalog` のデータベースが使用されます。</li><li>外部のカタログからデータベースを使用する場合は、このパラメータを指定する必要があります。詳細については、Example 2 を参照してください。</li><li>異なるカタログ間でデータベースを切り替える場合は、このパラメータを指定する必要があります。詳細については、Example 3 を参照してください。</li></ul>カタログについての詳細は、[概要](../../../data_source/catalog/catalog_overview.md)を参照してください。 |
| db_name        | はい      | データベース名。データベースは存在する必要があります。             |

## 使用例

Example 1: セッションのアクティブなデータベースとして `default_catalog` の `example_db` を使用します。

```SQL
USE default_catalog.example_db;
```

または

```SQL
USE example_db;
```

Example 2: セッションのアクティブなデータベースとして `hive_catalog` の `example_db` を使用します。

```SQL
USE hive_catalog.example_db;
```

Example 3: セッションのアクティブなデータベースを `hive_catalog.example_table1` から `iceberg_catalog.example_table2` に切り替えます。

```SQL
USE iceberg_catalog.example_table2;
```