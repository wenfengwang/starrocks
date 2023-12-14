---
displayed_sidebar: "Chinese"
---

# money_format

## 描述

此函数返回一个格式化为货币字符串的字符串。整数部分每三位用逗号分隔，小数部分保留两位。

## 语法

```Haskell
VARCHAR money_format(Number)
```

## 示例

```Plain Text
MySQL > select money_format(17014116);
+------------------------+
| money_format(17014116) |
+------------------------+
| 17,014,116.00          |
+------------------------+

MySQL > select money_format(1123.456);
+------------------------+
| money_format(1123.456) |
+------------------------+
| 1,123.46               |
+------------------------+

MySQL > select money_format(1123.4);
+----------------------+
| money_format(1123.4) |
+----------------------+
| 1,123.40             |
+----------------------+
```

## 关键词

MONEY_FORMAT,MONEY,FORMAT