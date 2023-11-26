---
displayed_sidebar: "Japanese"
---

# カタログ

## 説明

現在のカタログの名前を返します。カタログは、StarRocksの内部カタログまたは外部データソースにマップされた外部カタログのいずれかです。カタログについての詳細は、[カタログの概要](../../../data_source/catalog/catalog_overview.md)を参照してください。

カタログが選択されていない場合、StarRocksの内部カタログである `default_catalog` が返されます。

## 構文

```Haskell
catalog()
```

## パラメータ

この関数にはパラメータは必要ありません。

## 戻り値

現在のカタログの名前を文字列として返します。

## 例

例1: 現在のカタログはStarRocksの内部カタログである `default_catalog` です。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1 row in set (0.01 sec)
```

例2: 現在のカタログは外部カタログである `hudi_catalog` です。

```sql
-- 外部カタログに切り替える。
set catalog hudi_catalog;

-- 現在のカタログの名前を返す。
select catalog();
+--------------+
| CATALOG()    |
+--------------+
| hudi_catalog |
+--------------+
```

## 関連項目

[SET CATALOG](../../sql-statements/data-definition/SET_CATALOG.md): 宛先のカタログに切り替えます。
