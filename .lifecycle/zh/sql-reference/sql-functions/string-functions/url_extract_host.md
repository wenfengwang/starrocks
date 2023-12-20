---
displayed_sidebar: English
unlisted: true
---

# 提取URL中的主机部分

## 描述

从一个URL中提取出主机段。

## 语法

```haskell
VARCHAR url_extract_host(VARACHR str)
```

## 参数

- str：需要提取主机字符串的字符串。如果str不是字符串类型，该函数会首先尝试进行隐式类型转换。

## 返回值

返回提取出的主机字符串。

## 示例

```plaintext
mysql> select url_extract_host('httpa://starrocks.com/test/api/v1');
+-------------------------------------------------------+
| url_extract_host('httpa://starrocks.com/test/api/v1') |
+-------------------------------------------------------+
| starrocks.com                                         |
+-------------------------------------------------------+
```
