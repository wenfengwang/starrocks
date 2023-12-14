---
displayed_sidebar: "Chinese"
---

# min

## 描述

返回表达式 expr 的最小值。

## 语法

```Haskell
MIN(expr)
```

## 示例

```plain text
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