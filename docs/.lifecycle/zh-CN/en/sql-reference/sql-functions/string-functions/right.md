---
displayed_sidebar: "Chinese"
---

# right

## 描述

此函数返回给定字符串右侧指定长度的字符。长度单位：utf8字符。
注意：此函数也被称为[strright](strright.md)。

## 语法

```SQL
VARCHAR right(VARCHAR str, INT len)
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

## 关键词

RIGHT