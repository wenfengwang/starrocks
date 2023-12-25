---
displayed_sidebar: Chinese
---

# DOUBLE

## 説明

8バイトの浮動小数点数。

## 例

テーブル作成時にカラムの型をDOUBLEとして指定します。

```sql
CREATE TABLE doubleDemo (
    pk BIGINT(20) NOT NULL COMMENT "",
    income DOUBLE COMMENT "8バイト"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
