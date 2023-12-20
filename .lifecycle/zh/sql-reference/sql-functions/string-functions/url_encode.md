---
displayed_sidebar: English
---

# URL 编码

## 描述

将字符串转换成[application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1)格式。

该功能自 v3.2 版本起提供支持。

## 语法

```haskell
VARCHAR url_encode(VARCHAR str)
```

## 参数

- str：需要进行编码的字符串。如果 str 不是字符串类型，函数会先尝试进行隐式类型转换。

## 返回值

返回一个符合[application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1)格式的已编码字符串。

## 示例

```plaintext
mysql> select url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/');
+----------------------------------------------------------------------------+
| url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/') |
+----------------------------------------------------------------------------+
| https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F |
+----------------------------------------------------------------------------+
```
