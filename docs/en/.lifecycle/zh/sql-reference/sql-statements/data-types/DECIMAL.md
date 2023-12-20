---
displayed_sidebar: English
---

# DECIMAL

## 描述

DECIMAL(P[,S])

高精度定点数值。`P`代表总有效数字数（精度）。`S`代表小数点后的最大位数（刻度）。

如果省略`P`，默认值为10。如果省略`S`，默认值为0。

* Decimal V2

  `P`的取值范围是[1,27]，`S`的取值范围是[0,9]。`P`必须大于或等于`S`的值。`S`的默认值为0。

* 快速十进制（Decimal V3）

  `P`的取值范围是[1,38]，`S`的取值范围是[0, `P`]。`S`的默认值为0。Fast Decimal提供了更高的精度。

  主要优化：

  ​1. Fast Decimal使用可变宽度整数来表示小数。例如，对于精度小于或等于18的小数，它使用64位整数表示。而Decimal V2则对所有小数统一使用128位整数。在64位处理器上进行算术运算和转换操作时，使用的指令更少，这大大提高了性能。

  ​2.与Decimal V2相比，Fast Decimal在某些算法上进行了显著优化，特别是在乘法方面，性能提升了大约4倍。

Fast Decimal由FE动态参数`enable_decimal_v3`控制，默认值为`true`。

从v3.1版本开始，StarRocks支持在[ARRAY](Array.md)、[MAP](Map.md)和[STRUCT](STRUCT.md)中使用Fast Decimal类型。

## 限制

StarRocks不支持查询Hive中ORC和Parquet文件的DECIMAL数据。

## 示例

创建表时定义DECIMAL列。

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

decimal, Decimal V2, Decimal V3, Fast Decimal