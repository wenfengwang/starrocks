---
displayed_sidebar: Chinese
---

# VARCHAR

## 説明

VARCHAR(M)

可変長文字列。`M` は可変長文字列の長さを表し、単位はバイトで、デフォルト値は `1` です。

- StarRocks 2.1 以前のバージョンでは、`M` の値の範囲は 1~65533 です。
- 【パブリックベータ】StarRocks 2.1 バージョンから、`M` の値の範囲は 1~1048576 になります。

## 例

テーブル作成時にカラムの型を VARCHAR として指定します。

```sql
CREATE TABLE varcharDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "range char(m), m in (1-65533)"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```
