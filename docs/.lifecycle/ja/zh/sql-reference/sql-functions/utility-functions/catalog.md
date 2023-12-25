---
displayed_sidebar: Chinese
---

# カタログ

## 機能

現在のセッションが属するカタログを照会します。Internal CatalogまたはExternal Catalogのいずれかです。カタログの詳細については、[](../../../data_source/catalog/catalog_overview.md)を参照してください。

カタログが選択されていない場合、デフォルトではStarRocksシステムのInternal Catalog `default_catalog`が表示されます。

## 文法

```Haskell
catalog()
```

## パラメータ説明

この関数にはパラメータを渡す必要はありません。

## 戻り値の説明

現在のセッションが属するカタログ名を返します。

## 例

例1：現在のカタログはStarRocksシステムのInternal Catalogです。

```sql
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 row in set (0.01 sec)
```

例2：現在のカタログはExternal Catalog `hudi_catalog`です。

```sql
-- 目的のExternal Catalogに切り替えます。
set catalog hudi_catalog;

-- 現在のカタログ名を返します。
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 関連するSQL

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md)：指定されたカタログに切り替えます。
