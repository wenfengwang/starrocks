---
displayed_sidebar: English
---

# 解析 JSON

## 描述

将字符串转换成 JSON 值。

## 语法

```Haskell
parse_json(string_expr)
```

## 参数

string_expr：代表字符串的表达式。仅支持 STRING、VARCHAR 和 CHAR 数据类型。

## 返回值

返回一个 JSON 值。

> 注意：如果字符串不能被解析成一个标准的 JSON 值，PARSE_JSON 函数将返回 `NULL`（参见示例 5）。关于 JSON 规范的信息，请参考 [RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)。

## 示例

示例 1：把 STRING 类型的值 1 转换成 JSON 类型的值 1。

```plaintext
mysql> SELECT parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

示例 2：把 STRING 类型的数组转换成 JSON 数组。

```plaintext
mysql> SELECT parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

示例 3：把 STRING 类型的对象转换成 JSON 对象。

```plaintext
mysql> SELECT parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

示例 4：构造一个 JSON 类型的 NULL 值。

```plaintext
mysql> SELECT parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

示例 5：如果字符串不能被解析为标准的 JSON 值，PARSE_JSON 函数将返回 NULL。在这个例子中，星号没有被双引号（"）包围。因此，PARSE_JSON 函数返回 NULL。

```plaintext
mysql> SELECT parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## 关键字

parse_json，解析 JSON
