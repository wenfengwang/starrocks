---
displayed_sidebar: Chinese
---

# 使用方法

## 機能

セッションで使用するデータベースを指定します。データベースを指定した後、テーブルの作成やクエリの実行などの操作を行うことができます。

## 文法

```SQL
USE [<catalog_name>.]<db_name>
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| catalog_name   | 否       | カタログ名。<ul><li>このパラメータを指定しない場合、デフォルトで `default_catalog` のデータベースが使用されます。</li><li>外部カタログのデータベースを使用する場合は、このパラメータを指定する必要があります。詳細は「例2」を参照してください。</li><li>あるカタログのデータベースから別のカタログのデータベースに切り替える場合は、このパラメータを指定する必要があります。詳細は「例3」を参照してください。</li></ul>カタログに関する詳細は[概要](../../../data_source/catalog/catalog_overview.md)を参照してください。|
| db_name        | 是       | データベース名。指定されたデータベースは存在している必要があります。|

## 例

例1：`default_catalog` の `example_db` をセッションのデータベースとして使用します。

```SQL
USE default_catalog.example_db;
```

または

```SQL
USE example_db;
```

例2：`hive_catalog` の `example_db` をセッションのデータベースとして使用します。

```SQL
USE hive_catalog.example_db;
```

例3：セッションで使用しているデータベース `hive_catalog.example_table1` から `iceberg_catalog.example_table2` に切り替えます。

```SQL
USE iceberg_catalog.example_table2;
```
