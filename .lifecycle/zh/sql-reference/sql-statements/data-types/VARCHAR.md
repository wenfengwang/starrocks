---
displayed_sidebar: English
---

# VARCHAR

## 描述

VARCHAR(M)

一种可变长度的字符串。M 表示字符串的最大长度。默认值是 1。单位：字节。

- 在 StarRocks 2.1 之前的版本中，M 的取值范围是 1 到 65533。
- [预览] 从 StarRocks 2.1 及后续版本开始，M 的取值范围扩展为 1 到 1048576。

## 示例

创建一个表，并将列的类型指定为 VARCHAR。

```SQL
CREATE TABLE varcharDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    pd_type VARCHAR(20) COMMENT "range char(m),m in (1-65533) "
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```
