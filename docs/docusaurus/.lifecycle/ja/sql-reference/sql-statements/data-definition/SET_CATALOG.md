```yaml
---
displayed_sidebar: "Japanese"
---

# カタログの設定

現在のセッションで指定されたカタログに切り替えます。

このコマンドはv3.0以降でサポートされています。

> **注意**
>
> v3.1の新規展開StarRocksクラスターの場合、指定されたカタログに切り替えるには、そのカタログに対するUSAGE権限を持っている必要があります。[GRANT](../account-management/GRANT.md)を使用して必要な権限を付与することができます。以前のバージョンからアップグレードされたv3.1クラスターの場合は、継承された権限を使用してSET CATALOGを実行できます。

## 文法

```SQL
SET CATALOG <catalog_name>
```

## パラメーター

`catalog_name`: 現在のセッションで使用するカタログの名前。内部または外部カタログに切り替えることができます。指定したカタログが存在しない場合、例外がスローされます。

## 例

次のコマンドを実行して、現在のセッションで`hive_metastore`という名前のHiveカタログに切り替えます。

```SQL
SET CATALOG hive_metastore;
```

次のコマンドを実行して、現在のセッションで内部カタログ`default_catalog`に切り替えます。

```SQL
SET CATALOG default_catalog;
```