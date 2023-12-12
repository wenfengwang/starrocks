---
displayed_sidebar: "Japanese"
---

# money_format

## Description

この関数は、通貨の形式に整形された文字列を返します。整数部は3ビットごとにカンマで区切られ、小数部は2ビット分確保されます。

## Syntax

```Haskell
VARCHAR money_format(Number)
```

## Examples

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

## keyword

MONEY_FORMAT,MONEY,FORMAT