---
displayed_sidebar: English
---

# url_decode

## 描述

将字符串从 [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 格式转换回原格式。此函数是 [url_encode](./url_encode.md) 的逆操作。

该函数自 v3.2 版本起提供支持。

## 语法

```haskell
VARCHAR url_decode(VARCHAR str)
```

## 参数

- `str`：要解码的字符串。如果 `str` 不是字符串类型，系统将尝试进行隐式类型转换。

## 返回值

返回解码后的字符串。

## 示例

```plaintext
mysql> select url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F');
+------------------------------------------------------------------------------------------+
| url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F') |
+------------------------------------------------------------------------------------------+
| https://docs.starrocks.io/docs/introduction/StarRocks_intro/                             |
+------------------------------------------------------------------------------------------+
```