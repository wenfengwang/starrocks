---
displayed_sidebar: "英語"
---

# セット・カタログ

現在のセッションで指定されたカタログに切り替えます。

このコマンドはv3.0から対応されています。

> **注記**
>
> 新規デプロイされたStarRocks v3.1クラスターでは、指定した外部カタログにセットカタログして切り替えるにはUSAGE権限が必要です。必要な権限を付与するには[GRANT](../account-management/GRANT.md)を使用することができます。早いバージョンからアップグレードされたv3.1クラスターの場合、継承された権限を用いてSET CATALOGを実行できます。

## 構文

```SQL
SET CATALOG <catalog_name>
```

## パラメータ

`catalog_name`: 現在のセッションで使用するカタログの名前です。内部カタログまたは外部カタログに切り替えることができます。指定したカタログが存在しない場合、例外が発生します。

## 例

次のコマンドを実行して、現在のセッションでHiveカタログ`hive_metastore`に切り替えます：

```SQL
SET CATALOG hive_metastore;
```

次のコマンドを実行して、現在のセッションで内部カタログ`default_catalog`に切り替えます：

```SQL
SET CATALOG default_catalog;
```