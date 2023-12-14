---
displayed_sidebar: "Chinese"
---

# url_extract_host

## 描述

从url字符串中提取主机。

## 语法

```haskell
url_extract_host(str)
```

## 参数

- `str`：要提取其主机字符串的字符串。如果`str`不是字符串类型，将尝试进行隐式转换。

## 返回值

返回一个编码字符串。

## 示例

```plaintext
mysql> select url_extract_host('httpa://starrocks.com/test/api/v1');
+-------------------------------------------------------+
| url_extract_host('httpa://starrocks.com/test/api/v1') |
+-------------------------------------------------------+
| starrocks.com                                         |
+-------------------------------------------------------+
```