---
displayed_sidebar: Chinese
---

# json_object

## 機能

キーと値の集合を受け取り、それらのキーと値のペアを含むJSON型のオブジェクト（以下、JSONオブジェクトと呼ぶ）を返し、キーを辞書順に並べます。

## 文法

```Plain Text
JSON_OBJECT(key, value, ...)
```

## パラメータ説明

- `key`: JSONオブジェクトのキー。サポートされるデータ型はVARCHARです。

- `value`: JSONオブジェクトの値。サポートされるデータ型は文字列型（STRING、VARCHAR、CHAR）、JSON、数値型（TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT）、BOOLEAN、およびNULL値です。

## 戻り値の説明

JSONオブジェクトを返します。

> パラメータ`key`と`value`の合計が奇数の場合、最後のフィールドの値はnullで補完されます。

## 例

例1：複数のデータ型を含むJSONオブジェクトを構築します。

```Plain Text
mysql> SELECT JSON_OBJECT('name', 'starrocks', 'active', true, 'published', 2020);
       -> {"active": true, "name": "starrocks", "published": 2020}            
```

例2：複数のレベルにネストされたJSONオブジェクトを構築します。

```Plain Text
mysql> SELECT JSON_OBJECT('k1', 1, 'k2', JSON_OBJECT('k2', 2), 'k3', JSON_ARRAY(4, 5));
       -> {"k1": 1, "k2": {"k2": 2}, "k3": [4, 5]} 
```

例3：空のJSONオブジェクトを構築します。

```Plain Text
mysql> SELECT JSON_OBJECT();
       -> {}
```
