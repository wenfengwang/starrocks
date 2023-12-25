---
displayed_sidebar: Chinese
---

# BOOLEAN

## 説明

BOOL, BOOLEAN

TINYINTと同じように、0 は false を、1 は true を表します。

## 例

テーブル作成時にカラムの型を BOOLEAN として指定します。

```sql
CREATE TABLE booleanDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    ispass BOOLEAN COMMENT "true/false"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
