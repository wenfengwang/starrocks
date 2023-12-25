---
displayed_sidebar: English
---

# to_json

## 説明

Map または Struct の値を JSON 文字列に変換します。入力値が NULL の場合、NULL が返されます。

他のデータ型の値をキャストする場合は、[cast](./cast.md) を参照してください。

この関数は v3.1 以降でサポートされています。

## 構文

```Haskell
to_json(any_value)
```

## パラメーター

`any_value`: 変換したい Map または Struct の式です。入力値が無効な場合はエラーが返されます。Map または Struct の値の各キーと値のペアは nullable です。最後の例を参照してください。

## 戻り値

JSON 値を返します。

## 例

```Haskell
select to_json(map{1:'a',2:'b'});
+---------------------------+
| to_json(map{1:'a',2:'b'}) |
+---------------------------+
| {"1": "a", "2": "b"}      |
+---------------------------+

select to_json(row('asia','eu'));
+--------------------------------+
| to_json(row('asia', 'eu'))     |
+--------------------------------+
| {"col1": "asia", "col2": "eu"} |
+--------------------------------+

select to_json(map('a', named_struct('b', 1)));
+----------------------------------------+
| to_json(map{'a':named_struct('b', 1)}) |
+----------------------------------------+
| {"a": {"b": 1}}                        |
+----------------------------------------+

select to_json(named_struct("k1", cast(null as string), "k2", "v2"));
+-----------------------------------------------------------------------+
| to_json(named_struct('k1', CAST(NULL AS VARCHAR), 'k2', 'v2'))        |
+-----------------------------------------------------------------------+
| {"k1": null, "k2": "v2"}                                              |
+-----------------------------------------------------------------------+
```

## 関連項目

- [Map データ型](../../../sql-statements/data-types/Map.md)
- [Struct データ型](../../../sql-statements/data-types/STRUCT.md)
- [Map 関数](../../function-list.md#map-functions)
- [Struct 関数](../../function-list.md#struct-functions)
