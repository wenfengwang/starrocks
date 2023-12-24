---
displayed_sidebar: English
---

# MAX

## 描述

返回 expr 表达式的最大值。

## 语法

```Haskell
MAX(expr)
```

## 例子

```plain text
MySQL > select max(scan_rows)
from log_statis
group by datetime;
+------------------+
| max(`scan_rows`) |
+------------------+
|          4671587 |
+------------------+
```

## 关键词

MAX
