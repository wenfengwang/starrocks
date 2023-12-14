---
displayed_sidebar: "Chinese"
---

# 十进制

## 描述

DECIMAL(P[,S])

高精度定点值。`P` 代表有效数字的总位数（精度）。 `S` 代表小数点的最大位数（标度）。

如果省略 `P`，默认值为 10。如果省略 `S`，默认值为 0。

* 十进制 V2

  `P` 的范围为 [1,27]，`S` 的范围为 [0,9]。`P` 必须大于等于 `S` 的值。`S` 的默认值为 0。

* 快速十进制（十进制 V3）

  `P` 的范围为 [1,38]，`S` 的范围为 [0, P]。`S` 的默认值为 0。快速十进制提供了更高的精度。
  
  主要优化:
  
  1. 快速十进制使用可变宽度整数来表示小数。例如，对于精度小于或等于 18 的小数，它使用 64 位整数来表示小数。而十进制 V2 统一使用 128 位整数来表示所有小数。64 位处理器上的算术操作和转换操作使用更少的指令，大大提高了性能。
  
  2. 与十进制 V2 相比，快速十进制在一些算法中进行了重大优化，特别是在乘法方面，将性能提高了约 4 倍。

快速十进制由 FE 动态参数 `enable_decimal_v3` 控制，默认为 `true`。

从 v3.1 开始，StarRocks 支持 [ARRAY](Array.md)、[MAP](Map.md) 和 [STRUCT](STRUCT.md) 中的快速十进制条目。

## 限制

StarRocks 不支持在 Hive 的 ORC 和 Parquet 文件中查询 DECIMAL 数据。

## 示例

在创建表时定义 DECIMAL 列。

```sql
CREATE TABLE decimalDemo (
    pk BIGINT(20) NOT NULL COMMENT "",
    account DECIMAL(20,10) COMMENT ""
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);

INSERT INTO decimalDemo VALUES
(1,3.141592656),
(2,21.638378),
(3,4873.6293048479);

SELECT * FROM decimalDemo;
+------+-----------------+
| pk   | account         |
+------+-----------------+
|    1 |    3.1415926560 |
|    3 | 4873.6293048479 |
|    2 |   21.6383780000 |
+------+-----------------+
```

## 关键词

decimal, decimalv2, decimalv3, fast decimal