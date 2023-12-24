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

- `str`：要提取其主机字符串的字符串。如果 `str` 不是一个字符串，此函数将首先尝试隐式转换。

## 返回值

返回主机字符串。

## 例子

```plaintext
mysql> select url_extract_host('httpa://starrocks.com/test/api/v1');
+-------------------------------------------------------+
| url_extract_host('httpa://starrocks.com/test/api/v1') |
+-------------------------------------------------------+
| starrocks.com                                         |
+-------------------------------------------------------+