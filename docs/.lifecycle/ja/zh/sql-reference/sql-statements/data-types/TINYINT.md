---
displayed_sidebar: Chinese
---

# TINYINT

## 説明

1バイトの符号付き整数で、範囲は [-128, 127] です。

## 例

テーブル作成時にカラムの型をTINYINTとして指定します。

```sql
CREATE TABLE tinyIntDemo (
    pk TINYINT COMMENT "範囲 [-128, 127]",
    pd_type VARCHAR(20) COMMENT "範囲 char(m), mは(1-255)の間"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
