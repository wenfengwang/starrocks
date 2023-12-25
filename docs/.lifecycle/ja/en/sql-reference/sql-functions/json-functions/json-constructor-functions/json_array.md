---
displayed_sidebar: English
---

# json_array

## 説明

SQL 配列の各要素を JSON 値に変換し、その JSON 値から成る JSON 配列を返します。

## 構文

```Haskell
json_array(value, ...)
```

## パラメーター

`value`: SQL 配列の要素。`NULL` 値と以下のデータ型のみがサポートされています: STRING, VARCHAR, CHAR, JSON, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, DOUBLE, FLOAT, BOOLEAN。

## 戻り値

JSON 配列を返します。

## 例

例 1: 異なるデータ型の値で構成される JSON 配列を作成します。

```plaintext
mysql> SELECT json_array(1, true, 'starrocks', 1.1);

       -> [1, true, "starrocks", 1.1]
```

例 2: 空の JSON 配列を作成します。

```plaintext
mysql> SELECT json_array();

       -> []
```
