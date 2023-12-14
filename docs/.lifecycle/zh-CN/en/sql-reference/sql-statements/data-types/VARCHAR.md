---
displayed_sidebar: "Chinese"
---

# VARCHAR

## 描述

VARCHAR(M)

一个可变长度的字符串。`M` 表示字符串的长度。默认值为 `1`。单位：字节。

- 在StarRocks 2.1之前的版本中，`M` 的值范围为1–65533。
- [预览] 在StarRocks 2.1及更高版本中，`M` 的值范围为1–1048576。

## 示例

创建一个表并指定列类型为VARCHAR。

```SQL
CREATE TABLE varcharDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "范围 char(m)，m 在 (1-65533)"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```