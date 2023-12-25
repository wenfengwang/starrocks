---
displayed_sidebar: Chinese
---

# FLOAT

## 説明

4バイトの浮動小数点数。

## 例

テーブル作成時にカラムの型をFLOATとして指定します。

```sql
CREATE TABLE floatDemo (
    pk BIGINT(20) NOT NULL COMMENT "",
    channel FLOAT COMMENT "4バイト"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
