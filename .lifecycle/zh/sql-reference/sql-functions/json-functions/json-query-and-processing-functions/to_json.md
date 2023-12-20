---
displayed_sidebar: English
---

# to_json

## 描述

将 Map 或 Struct 值转换成 JSON 字符串。如果输入值为 NULL，将返回 NULL。

若需转换其它数据类型的值，请参见[cast](./cast.md)函数。

该功能自 v3.1 版本起提供支持。

## 语法

```Haskell
to_json(any_value)
```

## 参数

any_value：欲转换的 Map 或 Struct 表达式。若输入值无效，将返回错误信息。Map 或 Struct 值中的每个键值对的值都可以是可空的。详见最后一个示例。

## 返回值

返回一个 JSON 值。

## 示例

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

## 另请参阅

- [Map 数据类型](../../../sql-statements/data-types/Map.md)
- [Struct data type](../../../sql-statements/data-types/STRUCT.md) 数据类型
- [Map 函数](../../function-list.md#map-functions)
- [Struct 函数](../../function-list.md#struct-functions)
