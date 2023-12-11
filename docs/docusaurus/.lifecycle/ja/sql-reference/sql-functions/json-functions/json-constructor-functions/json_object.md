```yaml
---
displayed_sidebar: "Japanese"
---

# json_object

## Description

1つ以上のキーと値のペアを、キー値のペアで構成されるJSONオブジェクトに変換します。キー値のペアは辞書の順序でキーでソートされます。

## 構文

```Haskell
json_object(key, value, ...)
```

## パラメータ

- `key`: JSONオブジェクト内のキー。VARCHARデータ型のみがサポートされます。

- `value`: JSONオブジェクト内の値。`NULL`値および以下のデータ型がサポートされます：STRING、VARCHAR、CHAR、JSON、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT、およびBOOLEAN。

## 戻り値

JSONオブジェクトを返します。

> キーと値の合計数が奇数の場合、JSON_OBJECT関数は最後のフィールドに`NULL`を埋めます。

## 例

例1：異なるデータ型の値で構成されるJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object('name', 'starrocks', 'active', true, 'published', 2020);

       -> {"active": true, "name": "starrocks", "published": 2020}            
```

例2：ネストされたJSON_OBJECT関数を使用してJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object('k1', 1, 'k2', json_object('k2', 2), 'k3', json_array(4, 5));

       -> {"k1": 1, "k2": {"k2": 2}, "k3": [4, 5]} 
```

例3：空のJSONオブジェクトを作成します。

```plaintext
mysql> SELECT json_object();

       -> {}
```