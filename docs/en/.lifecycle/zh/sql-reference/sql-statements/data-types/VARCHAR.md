---
displayed_sidebar: English
---

# VARCHAR

## 描述

VARCHAR(M)

可变长度字符串。 `M` 表示字符串的长度。默认值为 `1`。单位：字节。

- 在 StarRocks 2.1 之前的版本中，`M` 的取值范围为 1–65533。
- [预览] 在 StarRocks 2.1 及之后的版本中，`M` 的取值范围为 1–1048576。

## 例子

创建一张表，并将列的类型指定为 VARCHAR。

```SQL
CREATE TABLE varcharDemo (
    pk INT COMMENT "范围 [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "范围 char(m)，m 为 (1-65533) "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```
