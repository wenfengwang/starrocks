---
displayed_sidebar: English
---

# 正确

## 描述

该函数能够从一个给定字符串的右侧截取指定长度的字符。长度单位为：UTF-8字符。
注意：此函数亦可被称作[strright](strright.md)。

## 语法

```SQL
VARCHAR right(VARCHAR str,INT len)
```

## 示例

```SQL
MySQL > select right("Hello starrocks",9);
+-----------------------------+
| right('Hello starrocks', 9) |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```

## 关键字

RIGHT
