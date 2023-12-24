---
displayed_sidebar: English
---

# lcase

## 描述

此函数将字符串转换为小写。它类似于lower函数。

## 语法

```Haskell
VARCHAR lcase(VARCHAR str)
```

## 例子

```Plain Text
mysql> SELECT lcase("AbC123");
+-----------------+
|lcase('AbC123')  |
+-----------------+
|abc123           |
+-----------------+
```

## 关键词

LCASE