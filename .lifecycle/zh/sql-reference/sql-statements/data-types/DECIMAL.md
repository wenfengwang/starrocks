---
displayed_sidebar: English
---

# 十进制

## 描述

DECIMAL(P[,S])

这是一个高精度的定点数值类型。P 表示总的有效数字位数（精度）。S 表示小数点后的最大位数（小数位数）。

如果省略 P，则默认值为 10。如果省略 S，则默认值为 0。

* 十进制 V2

  P 的取值范围为 [1,27]，S 的取值范围为 [0,9]。P 必须大于或等于 S 的值。S 的默认值为 0。

* 快速十进制（十进制 V3）

  P 的取值范围为 [1,38]，S 的取值范围为 [0,P]。S 的默认值为 0。快速十进制提供了更高的精确度。

  主要优化：

  1. 快速十进制使用可变宽度的整数来表示小数。例如，对于精度不超过 18 的小数，它使用 64 位整数表示。而十进制 V2 则统一对所有小数使用 128 位整数。在 64 位处理器上进行算术运算和转换操作时，指令数量更少，这大大提升了性能。

  2. 与十进制 V2 相比，快速十进制在某些算法上进行了显著优化，特别是在乘法运算上，性能提高了大约 4 倍。

快速十进制的使用由 FE 动态参数 enable_decimal_v3 控制，默认值为 true。

从 v3.1 版本开始，StarRocks支持在 [ARRAY](Array.md)、[MAP](Map.md) 和 [STRUCT](STRUCT.md) 数据结构中使用快速十进制类型。

## 限制

StarRocks 不支持查询存储在 Hive 中的 ORC 和 Parquet 文件格式的 DECIMAL 数据。

## 示例

在创建表时定义 DECIMAL 类型的列。

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

十进制、十进制 V2、十进制 V3、快速十进制
