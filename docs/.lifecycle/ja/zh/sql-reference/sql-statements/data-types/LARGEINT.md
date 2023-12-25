---
displayed_sidebar: Chinese
---

# LARGEINT

## 説明

16バイト符号付き整数、範囲は [-2^127 + 1 ~ 2^127 - 1]。

## 例

テーブル作成時にカラムの型をLARGEINTとして指定します。

```sql
CREATE TABLE largeIntDemo (
    pk LARGEINT COMMENT "range [-2^127 + 1 ~ 2^127 - 1]"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
