---
displayed_sidebar: Chinese
---

# SET CATALOG

## 機能

指定されたカタログに切り替えます。このコマンドはバージョン3.0からサポートされています。

> **注意**
>
> 新しくデプロイされた3.1クラスターの場合、ユーザーは目的のカタログのUSAGE権限を持っている必要があります。低いバージョンからアップグレードされたクラスターの場合は、権限を再度付与する必要はありません。[GRANT](../account-management/GRANT.md) コマンドを使用して権限を付与することができます。

## 文法

```SQL
SET CATALOG <catalog_name>
```

## パラメータ

`catalog_name`：現在のセッションで有効なカタログで、Internal CatalogとExternal Catalogがサポートされています。指定されたカタログが存在しない場合は、例外が発生します。

## 例

以下のコマンドを使用して、現在のセッションで有効なカタログをHive Catalogの`hive_metastore`に切り替えます：

```SQL
SET CATALOG hive_metastore;
```

以下のコマンドを使用して、現在のセッションで有効なカタログをInternal Catalogの`default_catalog`に切り替えます：

```SQL
SET CATALOG default_catalog;
```
