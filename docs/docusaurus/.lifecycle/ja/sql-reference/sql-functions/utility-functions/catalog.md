---
displayed_sidebar: "Japanese"
---

# カタログ

## 説明

現在のカタログの名前を返します。カタログは、StarRocks内部のカタログまたは外部データソースにマップされた外部カタログのいずれかです。カタログについての詳細は、[カタログの概要](../../../data_source/catalog/catalog_overview.md)を参照してください。

カタログが選択されていない場合、StarRocks内部のカタログ `default_catalog` が返されます。

## 構文

```Haskell
catalog()
```

## パラメータ

この関数にはパラメータは不要です。

## 戻り値

現在のカタログの名前を文字列で返します。

## 例

例1: 現在のカタログはStarRocks内部のカタログ `default_catalog` です。

```plaintext
select catalog();
+-----------------+
| CATALOG()       |
+-----------------+
| default_catalog |
+-----------------+
1行が返されました（0.01秒）
```

例2: 現在のカタログは外部カタログ `hudi_catalog` です。

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