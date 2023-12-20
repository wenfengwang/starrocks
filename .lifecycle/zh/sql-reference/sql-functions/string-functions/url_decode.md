---
displayed_sidebar: English
---

# URL 解码

## 描述

将字符串从 [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 格式反向转换。这个函数是 [url_encode](./url_encode.md) 的反向函数。

该函数从 v3.2 版本开始支持。

## 语法

```haskell
VARCAHR url_decode(VARCHAR str)
```

## 参数

- str：需要解码的字符串。如果 str 不是字符串类型，系统将会首先尝试进行隐式类型转换。

## 返回值

返回已解码的字符串。

## 示例

```plaintext
mysql> select url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F');
+------------------------------------------------------------------------------------------+
| url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F') |
+------------------------------------------------------------------------------------------+
| https://docs.starrocks.io/docs/introduction/StarRocks_intro/                             |
+------------------------------------------------------------------------------------------+
```
