---
displayed_sidebar: English
---

# url_encode

## 描述

将字符串转换为 [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 格式。

该函数从 v3.2 版本开始支持。

## 语法

```haskell
VARCHAR url_encode(VARCHAR str)
```

## 参数

- `str`：要编码的字符串。如果 `str` 不是字符串类型，该函数将尝试进行隐式类型转换。

## 返回值

返回一个符合 [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 格式的编码字符串。

## 示例

```plaintext
mysql> select url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/');
+----------------------------------------------------------------------------+
| url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/') |
+----------------------------------------------------------------------------+
| https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F |
+----------------------------------------------------------------------------+
```