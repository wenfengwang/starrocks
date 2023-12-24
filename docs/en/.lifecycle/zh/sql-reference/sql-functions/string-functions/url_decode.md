---
displayed_sidebar: English
---

# url_decode

## 描述

将字符串从 [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 格式转换回原始字符串。此函数是[url_encode](./url_encode.md)的逆向函数。

此函数从 v3.2 版本开始支持。

## 语法

```haskell
VARCHAR url_decode(VARCHAR str)
```

## 参数

- `str`：要解码的字符串。如果 `str` 不是一个字符串，系统将首先尝试进行隐式转换。

## 返回值

返回解码后的字符串。

## 例子

```plaintext
mysql> select url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F');
+------------------------------------------------------------------------------------------+
| url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F') |
+------------------------------------------------------------------------------------------+
| https://docs.starrocks.io/docs/introduction/StarRocks_intro/                             |
+------------------------------------------------------------------------------------------+