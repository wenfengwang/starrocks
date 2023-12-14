---
displayed_sidebar: "中文"
---

# PARSE_JSON

## 功能

将字符串类型的数据构造为 JSON 类型的数据。

## 语法

```Plain Text
PARSE_JSON(string_expr)
```

## 参数说明

`string_expr`: 字符串表达式。支持的数据类型有字符串类型（STRING、VARCHAR、CHAR）。

## 返回值说明

返回 JSON 类型的值。

> 如果字符串不能解析成标准的 JSON，则返回 NULL，详情见示例五。关于 JSON 标准，参见 [RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)。

## 示例

示例一：将字符串类型的 1 转换成 JSON 类型的 1。

```Plain Text
mysql> SELECT PARSE_JSON('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

示例二：将一个字符串类型的数组转换成一个 JSON 类型的数组。

```Plain Text
mysql> SELECT PARSE_JSON('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

示例三：将一个字符串类型的对象转换成一个 JSON 类型的对象。

```Plain Text
mysql> SELECT PARSE_JSON('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

示例四：构造一个 JSON 类型的 NULL。

```Plain Text
mysql> SELECT PARSE_JSON('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

示例五：如果字符串无法被解析成标准的 JSON，那么会返回 NULL。在下面的例子中，star 没有被双引号包围，因此无法被解析成有效的 JSON，结果返回 NULL。

```Plain Text
mysql> SELECT PARSE_JSON('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## 关键字

parse_json, parse json