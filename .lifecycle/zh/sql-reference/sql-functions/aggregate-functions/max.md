---
displayed_sidebar: English
---

# 最大值

## 描述

返回表达式 expr 的最大值。

## 语法

```Haskell
MAX(expr)
```

## 示例

```plain
MySQL > select max(scan_rows)
from log_statis
group by datetime;
+------------------+
| max(`scan_rows`) |
+------------------+
|          4671587 |
+------------------+
```

## 关键字

MAX
