---
displayed_sidebar: "Chinese"
---

# parse_json

## 描述

将字符串转换为JSON值。

## 语法

```Haskell
parse_json(string_expr)
```

## 参数

`string_expr`：表示字符串的表达式。仅支持STRING、VARCHAR和CHAR数据类型。

## 返回值

返回一个JSON值。

> 注意：如果字符串无法解析为标准的JSON值，PARSE_JSON函数将返回`NULL`（参见示例5）。有关JSON规范的信息，请参阅[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)。

## 示例

示例1：将STRING值`1`转换为JSON值`1`。

```plaintext
mysql> SELECT parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

示例2：将STRING数据类型的数组转换为JSON数组。

```plaintext
mysql> SELECT parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

示例3：将STRING数据类型的对象转换为JSON对象。

```plaintext
mysql> SELECT parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

示例4：构造一个JSON值为`NULL`。

```plaintext
mysql> SELECT parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

示例5：如果字符串无法解析为标准的JSON值，PARSE_JSON函数将返回`NULL`。在此示例中，`star`没有用双引号（"）括起来。因此，PARSE_JSON函数返回`NULL`。

```plaintext
mysql> SELECT parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## 关键词

parse_json, parse json