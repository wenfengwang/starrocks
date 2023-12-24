---
displayed_sidebar: English
---

# parse_url

## 描述

解析 URL 并从中提取组件。

## 语法

```Haskell
parse_url(expr1, expr2);
```

## 参数

`expr1`：URL。支持的数据类型为 VARCHAR。

`expr2`：要从此 URL 中提取的组件。支持的数据类型为 VARCHAR。有效值：

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。无法返回 QUERY 中的参数。如果要返回特定参数，请使用 [trim](trim.md) 结合 parse_url 来实现。详情请参阅[示例](#examples)。

`expr2` 区分 **大小写**。

## 返回值

返回 VARCHAR 类型的值。如果 URL 无效，则返回错误。如果找不到请求的信息，则返回 NULL。

## 例子

```Plain Text
select parse_url('http://facebook.com/path/p1.php?query=1', 'HOST');
+--------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'HOST') |
+--------------------------------------------------------------+
| facebook.com                                                 |
+--------------------------------------------------------------+

select parse_url('http://facebook.com/path/p1.php?query=1', 'AUTHORITY');
+-------------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'AUTHORITY') |
+-------------------------------------------------------------------+
| facebook.com                                                      |
+-------------------------------------------------------------------+

select parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY');
+---------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY') |
+---------------------------------------------------------------+
| query=1                                                       |
+---------------------------------------------------------------+

select trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'), 'query='); 
+-------------------------------------------------------------------------------+
| trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'), 'query=') |
+-------------------------------------------------------------------------------+
| 1                                                                             |
+-------------------------------------------------------------------------------+