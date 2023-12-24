---
displayed_sidebar: English
---

# parse_json

## 描述

将字符串转换为 JSON 值。

## 语法

```Haskell
parse_json(string_expr)
```

## 参数

`string_expr`：表示字符串的表达式。仅支持 STRING、VARCHAR 和 CHAR 数据类型。

## 返回值

返回 JSON 值。

> 注意：如果字符串无法解析为标准 JSON 值，则 PARSE_JSON 函数将返回 `NULL` （请参阅示例 5）。有关 JSON 规范的信息，请参阅 [RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)。

## 例子

示例 1：将字符串值为 `1` 的 STRING 转换为 JSON 值为 `1`。

```plaintext
mysql> SELECT parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

示例 2：将 STRING 数据类型的数组转换为 JSON 数组。

```plaintext
mysql> SELECT parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

示例 3：将 STRING 数据类型的对象转换为 JSON 对象。

```plaintext
mysql> SELECT parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

示例 4：构造 JSON 值为 `NULL`。

```plaintext
mysql> SELECT parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

示例 5：如果无法将字符串解析为标准 JSON 值，则 PARSE_JSON 函数将返回 `NULL`。在此示例中，`star` 未用双引号（"）括起来。因此，PARSE_JSON 函数返回 `NULL`。

```plaintext
mysql> SELECT parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## 关键字

parse_json，解析json
