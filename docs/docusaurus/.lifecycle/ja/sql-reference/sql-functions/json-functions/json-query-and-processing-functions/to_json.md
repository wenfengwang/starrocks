---
displayed_sidebar: "日本語"
---

# to_json

## 説明

MapまたはStructの値をJSON文字列に変換します。入力値がNULLの場合、NULLが返されます。

他のデータ型の値をキャストしたい場合は、[cast](./cast.md)を参照してください。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
to_json(any_value)
```

## パラメータ

`any_value`: 変換したいMapまたはStructの式。入力値が無効な場合、エラーが返されます。MapまたはStructの値のキーと値のペアごとの値はnullableです。最後の例を参照してください。

## 戻り値

JSON値を返します。

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
| to_json(named_struct('k1', CAST(NULL AS VARCHAR(65533)), 'k2', 'v2')) |
+-----------------------------------------------------------------------+
| {"k1": null, "k2": "v2"}                                              |
+-----------------------------------------------------------------------+
```

## 関連項目

- [Mapデータ型](../../../sql-statements/data-types/Map.md)
- [Structデータ型](../../../sql-statements/data-types/STRUCT.md)
- [Map関数](../../function-list.md#map-functions)
- [Struct関数](../../function-list.md#struct-functions)