---
displayed_sidebar: Chinese
---

# カタログを表示

## 機能

現在のクラスターにあるすべてのカタログを表示します。これには、Internal Catalog と External Catalog が含まれます。

> **注意**
>
> External Catalog USAGE 権限を持つユーザーのみがそのカタログを表示できます。その権限がない場合は、`default_catalog` のみが返されます。権限を付与するには [GRANT](../account-management/GRANT.md) コマンドを使用してください。

## 文法

```SQL
SHOW CATALOGS
```

## 返り値の説明

```SQL
+----------+--------+----------+
| Catalog  | Type   | Comment  |
+----------+--------+----------+
```

返り値のフィールドの説明は以下の通りです：

| **フィールド** | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| Catalog        | カタログ名。                                                 |
| Type           | カタログのタイプ。`default_catalog` の場合は `Internal` が返されます。External Catalog の場合は、そのタイプが返されます。例えば `Hive`, `Hudi`, `Iceberg` などです。 |
| Comment        | カタログのコメント。<ul><li>External Catalog を作成する際にコメントを追加することはできません。そのため、External Catalog の場合、`Comment` は `NULL` として返されます。</li><li>`default_catalog` の場合、デフォルトで `Comment` は `An internal catalog contains this cluster's self-managed tables.` として返されます。`default_catalog` は StarRocks クラスター内で唯一の Internal Catalog であり、削除することはできません。</li></ul> |

## 例

現在のクラスターにあるすべてのカタログを表示します。

```SQL
SHOW CATALOGS\G
*************************** 1. row ***************************
Catalog: default_catalog
   Type: Internal
Comment: An internal catalog contains this cluster's self-managed tables.
*************************** 2. row ***************************
Catalog: hudi_catalog
   Type: Hudi
Comment: NULL
*************************** 3. row ***************************
Catalog: iceberg_catalog
   Type: Iceberg
Comment: NULL
```
