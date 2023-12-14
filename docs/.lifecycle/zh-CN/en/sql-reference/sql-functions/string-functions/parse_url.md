```markdown
---
displayed_sidebar: "Chinese"
---

# parse_url

## 描述

解析URL并从中提取一个组件。

## 语法

```Haskell
parse_url(expr1,expr2);
```

## 参数

`expr1`：URL。支持的数据类型为VARCHAR。

`expr2`：要从此URL中提取的组件。支持的数据类型为VARCHAR。有效值：

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。QUERY中的参数无法返回。如果要返回特定参数，请使用[trim](trim.md)结合parse_url来实现。有关详细信息，请参见[示例](#示例)。

`expr2` **区分大小写**。

## 返回值

返回VARCHAR类型的值。如果URL无效，则返回错误。如果无法找到请求的信息，则返回NULL。

## 示例

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

select trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'),'query='); 
+-------------------------------------------------------------------------------+
| trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'), 'query=') |
+-------------------------------------------------------------------------------+
| 1                                                                             |
+-------------------------------------------------------------------------------+
```