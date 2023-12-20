---
displayed_sidebar: English
---

# lcase

## 描述

此函数将字符串转换为小写。它与 lower 函数功能相似。

## 语法

```Haskell
VARCHAR lcase(VARCHAR str)
```

## 示例

```Plain
mysql> SELECT lcase("AbC123");
+-----------------+
|lcase('AbC123')  |
+-----------------+
|abc123           |
+-----------------+
```

## 关键字

LCASE