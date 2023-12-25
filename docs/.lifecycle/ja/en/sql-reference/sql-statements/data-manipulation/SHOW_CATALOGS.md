---
displayed_sidebar: English
---

# SHOW CATALOGS

## 説明

現在のStarRocksクラスターにある全てのカタログを照会します。これには内部カタログと外部カタログが含まれます。

> **注記**
>
> SHOW CATALOGSは、外部カタログに対するUSAGE権限を持つユーザーに外部カタログを返します。ユーザーやロールがいずれの外部カタログにもこの権限を持っていない場合、このコマンドはdefault_catalogのみを返します。

## 構文

```SQL
SHOW CATALOGS
```

## 出力

```SQL
+----------+--------+----------+
| Catalog  | Type   | Comment  |
+----------+--------+----------+
```

次の表は、このステートメントで返されるフィールドを説明しています。

| **フィールド** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| Catalog       | カタログ名です。                                            |
| Type          | カタログのタイプです。カタログが`default_catalog`の場合は`Internal`が返されます。カタログが外部カタログの場合は、`Hive`、`Hudi`、`Iceberg`などの対応するカタログタイプが返されます。 |
| Comment       | カタログのコメントです。StarRocksは外部カタログへのコメント追加をサポートしていません。そのため、外部カタログの場合は値が`NULL`になります。カタログが`default_catalog`の場合、デフォルトでコメントは`An internal catalog contains this cluster's self-managed tables.`となります。`default_catalog`はStarRocksクラスター内の唯一の内部カタログです。 |

## 例

現在のクラスター内の全てのカタログを照会します。

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
