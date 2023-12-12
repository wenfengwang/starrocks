---
displayed_sidebar: "Japanese"
---

# カタログの表示

## 説明

現在のStarRocksクラスターですべてのカタログを問い合わせます。内部カタログと外部カタログを含みます。

> **注意**
>
> SHOW CATALOGSは、その外部カタログにUSAGE権限を持つユーザーに外部カタログを返します。外部カタログへのUSAGE権限がユーザーやロールにない場合、このコマンドはdefault_catalogのみを返します。

## 構文

```SQL
SHOW CATALOGS
```

## 出力

```SQL
+----------+--------+----------+
| カタログ  | タイプ | コメント |
+----------+--------+----------+
```

次の表は、この文によって返されるフィールドを説明します。

| **フィールド** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| カタログ       | カタログ名。                                            |
| タイプ          | カタログのタイプ。カタログが`default_catalog`の場合は`Internal`が返されます。`Hive`、`Hudi`、`Iceberg`などの外部カタログの場合、対応するカタログタイプが返されます。 |
| コメント       | カタログのコメント。StarRocksは外部カタログにコメントを追加することをサポートしていません。したがって、外部カタログの場合、値は`NULL`です。カタログが`default_catalog`の場合は、コメントはデフォルトで`An internal catalog contains this cluster's self-managed tables.`です。`default_catalog`はStarRocksクラスターの唯一の内部カタログです。|

## 例

現在のクラスターですべてのカタログを問い合わせます。

```SQL
SHOW CATALOGS\G
*************************** 1. 行 ***************************
カタログ: default_catalog
   タイプ: Internal
コメント: An internal catalog contains this cluster's self-managed tables.
*************************** 2. 行 ***************************
カタログ: hudi_catalog
   タイプ: Hudi
コメント: NULL
*************************** 3. 行 ***************************
カタログ: iceberg_catalog
   タイプ: Iceberg
コメント: NULL
```