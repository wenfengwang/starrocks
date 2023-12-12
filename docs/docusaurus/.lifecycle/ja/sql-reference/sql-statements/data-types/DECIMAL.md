---
displayed_sidebar: "English"
---

# DECIMAL（小数点数）

## 説明

DECIMAL(P[,S])

高精度の固定小数点値。 `P` は有効数字の総数（精度）を示し、 `S` は小数点以下の最大桁数（スケール）を示します。
`P` を省略すると、デフォルト値は10です。 `S` を省略すると、デフォルト値は0です。

* Decimal V2（小数点V2）

  `P` の範囲は [1,27] で、 `S` の範囲は [0,9] です。 `P` は `S` の値以上でなければなりません。`S`のデフォルト値は0です。

* Fast Decimal（小数点V3）

  `P` の範囲は [1,38] で、 `S` の範囲は [0, P] です。 `S` のデフォルト値は0です。Fast Decimalはより高い精度を提供します。
  
  主な最適化点：
  
  1. Fast Decimalは可変幅整数を使用して小数を表現します。たとえば、精度が18以下の小数には64ビット整数を使用します。一方、Decimal V2ではすべての小数に対して一律に128ビット整数を使用します。64ビットプロセッサ上での算術演算および変換操作は、命令数が少なくなり、性能が大幅に向上します。
  
  2. Decimal V2と比較して、Fast Decimalは一部のアルゴリズムにおいて特に乗算において大幅な最適化を行い、性能を約4倍向上させました。

Fast Decimalはデフォルトで`true`であるFE動的パラメータ`enable_decimal_v3`によってコントロールされます。

v3.1以降、StarRocksは [ARRAY](Array.md)、[MAP](Map.md)、および [STRUCT](STRUCT.md) でFast Decimalエントリをサポートします。

## 制限

StarRocksはHiveからORCおよびParquetファイル内のDECIMALデータのクエリをサポートしていません。

## 例

テーブルを作成する際にDECIMAL列を定義します。

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

## キーワード

decimal, decimalv2, decimalv3, fast decimal