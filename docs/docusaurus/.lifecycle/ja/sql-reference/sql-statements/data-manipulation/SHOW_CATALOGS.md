---
displayed_sidebar: "Japanese"
---

# カタログを表示

## 説明

現在のStarRocksクラスターにあるすべてのカタログ、内部カタログと外部カタログを問い合わせます。

> **注**
>
> SHOW CATALOGSは、外部カタログのUSAGE権限を持つユーザーに外部カタログを返します。ユーザーまたはロールがどの外部カタログにもこの権限を持っていない場合、このコマンドはdefault_catalogのみを返します。

## 構文

```SQL
SHOW CATALOGS
```

## 出力

```SQL
+----------+--------+----------+
| カタログ  | タイプ   | コメント  |
+----------+--------+----------+
```

次の表は、このステートメントによって返されるフィールドを説明しています。

| **フィールド** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| カタログ       | カタログ名。                                            |
| タイプ          | カタログのタイプ。`default_catalog`の場合は`Internal`が返されます。他の外部カタログの場合は、`Hive`、`Hudi`、または`Iceberg`などの対応するカタログタイプが返されます。 |
| コメント       | カタログのコメント。StarRocksは外部カタログへのコメントの追加をサポートしていないため、外部カタログの場合は値が`NULL`になります。`default_catalog`の場合、コメントはデフォルトで`An internal catalog contains this cluster's self-managed tables.`になります。`default_catalog`は、StarRocksクラスターにおける唯一の内部カタログです。 |

## 例

現在のクラスターにあるすべてのカタログを問い合わせる。

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