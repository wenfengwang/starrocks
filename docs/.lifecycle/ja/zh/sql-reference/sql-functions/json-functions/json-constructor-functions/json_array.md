---
displayed_sidebar: Chinese
---

# json_array

## 機能

SQL 配列を受け取り、JSON 形式の配列（以下、JSON 配列と呼ぶ）を返します。

## 文法

```Plain Text
JSON_ARRAY(value, ...)
```

## パラメータ説明

`value`: 配列の要素です。サポートされるデータ型は文字列型（STRING、VARCHAR、CHAR）、JSON、数値型（TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT）、BOOLEAN、および NULL 値です。

## 戻り値の説明

JSON 配列を返します。

## 例

例1：様々なデータ型で構成される JSON 配列を作成します。

```Plain Text
mysql> SELECT JSON_ARRAY(1, true, 'starrocks', 1.1);
       -> [1, true, "starrocks", 1.1]
```

例2：空の JSON 配列を作成します。

```Plain Text
mysql> SELECT JSON_ARRAY();
       -> []
```
