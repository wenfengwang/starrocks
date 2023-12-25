---
displayed_sidebar: Chinese
---

# min

## 機能

`expr` 式の最小値を返します。

## 文法

```Haskell
MIN(expr)
```

## 引数説明

`expr`: 選択される式。

## 戻り値の説明

数値型の戻り値を返します。

## 例

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
