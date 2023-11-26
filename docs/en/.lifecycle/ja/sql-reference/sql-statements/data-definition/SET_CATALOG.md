---
displayed_sidebar: "Japanese"
---

# カタログの設定

現在のセッションで指定されたカタログに切り替えます。

このコマンドはv3.0以降でサポートされています。

> **注意**
>
> 新しく展開されたStarRocks v3.1クラスターでは、SET CATALOGを実行してそのカタログに切り替える場合、宛先の外部カタログにUSAGE権限が必要です。必要な権限を付与するには、[GRANT](../account-management/GRANT.md)を使用できます。以前のバージョンからアップグレードされたv3.1クラスターでは、継承された権限でSET CATALOGを実行できます。

## 構文

```SQL
SET CATALOG <catalog_name>
```

## パラメータ

`catalog_name`: 現在のセッションで使用するカタログの名前です。内部または外部のカタログに切り替えることができます。指定したカタログが存在しない場合、例外がスローされます。

## 例

次のコマンドを実行して、現在のセッションでHiveカタログの名前が`hive_metastore`のカタログに切り替えます。

```SQL
SET CATALOG hive_metastore;
```

次のコマンドを実行して、現在のセッションで内部カタログ`default_catalog`に切り替えます。

```SQL
SET CATALOG default_catalog;
```
