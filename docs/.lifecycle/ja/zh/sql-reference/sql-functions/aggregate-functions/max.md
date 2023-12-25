---
displayed_sidebar: Chinese
---

# max

## 機能

`expr` 式の最大値を返します。

## 文法

```Haskell
MAX(expr)
```

## 引数説明

`expr`: 選択された式。

## 戻り値の説明

数値型の戻り値を返します。

## 例

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
