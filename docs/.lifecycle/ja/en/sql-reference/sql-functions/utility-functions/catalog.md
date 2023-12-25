---
displayed_sidebar: English
---

# カタログ

## 説明

現在のカタログの名前を返します。カタログはStarRocksの内部カタログ、または外部データソースにマッピングされた外部カタログです。カタログについての詳細は、[カタログ概要](../../../data_source/catalog/catalog_overview.md)をご覧ください。

カタログが選択されていない場合、StarRocksの内部カタログ`default_catalog`が返されます。

## 構文

```Haskell
catalog()
```

## パラメーター

この関数はパラメーターを必要としません。

## 戻り値

現在のカタログの名前を文字列で返します。

## 例

例 1: 現在のカタログはStarRocksの内部カタログ`default_catalog`です。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 row in set (0.01 sec)
```

例 2: 現在のカタログは外部カタログ`hudi_catalog`です。

```sql
-- 外部カタログに切り替えます。
set catalog hudi_catalog;

-- 現在のカタログの名前を返します。
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 関連項目

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md): 宛先カタログに切り替えます。
