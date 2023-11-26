---
displayed_sidebar: "Japanese"
---

# json_array

## 説明

SQL配列の各要素をJSON値に変換し、JSON値で構成されるJSON配列を返します。

## 構文

```Haskell
json_array(value, ...)
```

## パラメータ

`value`: SQL配列の要素です。NULL値と以下のデータ型のみサポートされます：STRING、VARCHAR、CHAR、JSON、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT、およびBOOLEAN。

## 戻り値

JSON配列を返します。

## 例

例1：異なるデータ型の値で構成されるJSON配列を作成します。

```plaintext
mysql> SELECT json_array(1, true, 'starrocks', 1.1);

       -> [1, true, "starrocks", 1.1]
```

例2：空のJSON配列を作成します。

```plaintext
mysql> SELECT json_array();

       -> []
```
