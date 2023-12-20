---
displayed_sidebar: English
---

# 解析网址

## 描述

解析一个URL，并从中提取特定的组件。

## 语法

```Haskell
parse_url(expr1,expr2);
```

## 参数

expr1：URL。支持的数据类型为VARCHAR。

expr2：要从URL中提取的组件。支持的数据类型为VARCHAR。有效的值包括：

- PROTOCOL（协议）
- HOST（主机）
- PATH（路径）
- REF（引用）
- AUTHORITY（权限）
- FILE（文件）
- USERINFO（用户信息）
- QUERY. Parameters in QUERY（查询） cannot be returned. If you want to return specific parameters, use parse_url with [trim](trim.md) to achieve this implementation. For details, see [Examples](#examples).

`expr2` 对**大小写**敏感。

## 返回值

返回VARCHAR类型的值。如果URL无效，则返回错误信息。如果无法找到请求的信息，则返回NULL。

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
