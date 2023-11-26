---
displayed_sidebar: "Japanese"
---

# json_object

## 説明

キーと値のペアを1つ以上指定して、キーと値のペアから構成されるJSONオブジェクトに変換します。キーと値のペアは、辞書順にキーでソートされます。

## 構文

```Haskell
json_object(key, value, ...)
```

## パラメータ

- `key`: JSONオブジェクトのキーです。VARCHARデータ型のみサポートされています。

- `value`: JSONオブジェクトの値です。`NULL`値と以下のデータ型のみサポートされています: STRING、VARCHAR、CHAR、JSON、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT、BOOLEAN。

## 戻り値

JSONオブジェクトを返します。

> キーと値の総数が奇数の場合、JSON_OBJECT関数は最後のフィールドに`NULL`を埋めます。

## 例

例1: 異なるデータ型の値から構成されるJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object('name', 'starrocks', 'active', true, 'published', 2020);

       -> {"active": true, "name": "starrocks", "published": 2020}            
```

例2: ネストされたJSON_OBJECT関数を使用してJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object('k1', 1, 'k2', json_object('k2', 2), 'k3', json_array(4, 5));

       -> {"k1": 1, "k2": {"k2": 2}, "k3": [4, 5]} 
```

例3: 空のJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object();

       -> {}
```
