---
displayed_sidebar: English
---

# 案例

## 描述

此函数可将字符串转换为小写形式。其功能与 lower 函数相似。

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
