---
displayed_sidebar: English
unlisted: true
---

# url_extract_host

## 描述

从 URL 中提取主机部分。

## 语法

```haskell
VARCHAR url_extract_host(VARCHAR str)
```

## 参数

- `str`：要从中提取主机字符串的字符串。如果 `str` 不是字符串类型，此函数会尝试进行隐式类型转换。

## 返回值

返回主机字符串。

## 示例

```plaintext
mysql> select url_extract_host('httpa://starrocks.com/test/api/v1');
+-------------------------------------------------------+
| url_extract_host('httpa://starrocks.com/test/api/v1') |
+-------------------------------------------------------+
| starrocks.com                                         |
+-------------------------------------------------------+
```