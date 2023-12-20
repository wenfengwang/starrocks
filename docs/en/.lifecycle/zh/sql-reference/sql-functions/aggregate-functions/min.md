---
displayed_sidebar: English
---

# min

## 描述

返回 expr 表达式的最小值。

## 语法

```Haskell
MIN(expr)
```

## 示例

```plain
MySQL > select min(scan_rows)
from log_statis
group by datetime;
+------------------+
| min(`scan_rows`) |
+------------------+
|                0 |
+------------------+
```

## 关键字

MIN