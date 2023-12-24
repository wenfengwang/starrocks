---
displayed_sidebar: English
---

# RIGHT

## 描述

此函数从给定字符串的右侧返回指定长度的字符。长度单位：utf8 字符。
注意：此函数也被称为 [strright](strright.md)。

## 语法

```SQL
VARCHAR right(VARCHAR str,INT len)
```

## 例子

```SQL
MySQL > select right("Hello starrocks",9);
+-----------------------------+
| right('Hello starrocks', 9) |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```

## 关键词

RIGHT