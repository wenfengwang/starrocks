---
displayed_sidebar: English
---

# ucase

## 描述

此函数将字符串转换为大写。它类似于函数 upper。

## 语法

```Haskell
VARCHAR ucase(VARCHAR str)
```

## 例子

```Plain Text
mysql> SELECT ucase("AbC123");
+-----------------+
|ucase('AbC123')  |
+-----------------+
|ABC123           |
+-----------------+
```

## 关键词

UCASE