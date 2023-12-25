---
displayed_sidebar: English
---

# SET CATALOG

現在のセッションで指定されたカタログに切り替えます。

このコマンドはv3.0以降でサポートされています。

> **注記**
>
> 新規にデプロイされたStarRocks v3.1クラスタでは、SET CATALOGを使用して特定のカタログに切り替えるためには、その外部カタログに対するUSAGE権限が必要です。必要な権限を付与するには[GRANT](../account-management/GRANT.md)を使用してください。以前のバージョンからアップグレードされたv3.1クラスタでは、既にある権限を引き継いでSET CATALOGを実行できます。

## 構文

```SQL
SET CATALOG <catalog_name>
```

## パラメータ

`catalog_name`: 現在のセッションで使用するカタログの名前です。内部カタログまたは外部カタログに切り替えることができます。指定されたカタログが存在しない場合、例外が投げられます。

## 例

次のコマンドを実行して、現在のセッションで`hive_metastore`という名前のHiveカタログに切り替えます。

```SQL
SET CATALOG hive_metastore;
```

次のコマンドを実行して、現在のセッションで内部カタログ`default_catalog`に切り替えます。

```SQL
SET CATALOG default_catalog;
```
