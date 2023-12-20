---
displayed_sidebar: English
---

# parse_url

## 描述

解析一个URL并从中提取一个组件。

## 语法

```Haskell
parse_url(expr1,expr2);
```

## 参数

`expr1`：URL。支持的数据类型是 VARCHAR。

`expr2`：要从该URL中提取的组件。支持的数据类型是 VARCHAR。有效值包括：

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。QUERY中的参数不能被直接返回。如果您想返回特定参数，请结合使用parse_url和[trim](trim.md)来实现。详情请见[示例](#examples)。

`expr2` 是**区分大小写**的。

## 返回值

返回VARCHAR类型的值。如果URL无效，则返回错误。如果无法找到请求的信息，则返回NULL。

## 示例

```Plain
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

select trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'),'query='); 
+-------------------------------------------------------------------------------+
| trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'), 'query=') |
+-------------------------------------------------------------------------------+
| 1                                                                             |
+-------------------------------------------------------------------------------+
```