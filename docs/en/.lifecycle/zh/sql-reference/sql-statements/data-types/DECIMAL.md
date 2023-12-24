---
displayed_sidebar: English
---

# 十进制

## 描述

DECIMAL(P[,S])

高精度定点值。`P`代表有效数字的总数（精度）。`S`代表最大小数位数（刻度）。

如果省略`P`，默认值为10。如果省略`S`，默认值为0。

* 十进制 V2

  `P`的范围是[1,27]，`S`的范围是[0,9]。`P`必须大于等于`S`的值。`S`的默认值为0。

* 快速十进制（Decimal V3）

  `P`的范围是[1,38]，`S`的范围是[0, P]。`S`的默认值为0。快速十进制提供了更高的精度。
  
  主要优化：
  
  1. 快速十进制使用可变宽度整数来表示小数。例如，它使用64位整数来表示精度小于或等于18的小数。而十进制 V2 对所有小数统一使用128位整数。64位处理器上的算术运算和转换运算使用的指令更少，这大大提高了性能。
  
  2. 与十进制 V2 相比，快速十进制在一些算法上做了显著的优化，特别是在乘法方面，性能提升了约4倍。

快速十进制由 FE 动态参数 `enable_decimal_v3` 控制，默认值为 `true`。

从v3.1开始，StarRocks支持 [ARRAY](Array.md)、[MAP](Map.md) 和 [STRUCT](STRUCT.md) 中的快速十进制条目。

## 限制

StarRocks不支持在Hive中查询ORC和Parquet文件中的DECIMAL数据。

## 例子

在创建表时定义DECIMAL列。

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

十进制、十进制V2、十进制V3、快速十进制
