---
displayed_sidebar: "Japanese"
---

# DECIMAL（デシマル）

## 説明

DECIMAL(P[,S])

高精度固定小数点値。 `P` は有効数字（精度）の総数を示します。 `S` は小数点以下の最大桁数（スケール）を示します。

`P` が省略された場合、デフォルトは10です。 `S` が省略された場合、デフォルトは0です。

* Decimal V2

  `P` の範囲は [1,27] で、`S` の範囲は [0,9] です。 `P` は `S` の値以上でなければなりません。 `S` のデフォルト値は0です。

* Fast Decimal（Decimal V3）

  `P` の範囲は [1,38] で、`S` の範囲は [0, P] です。 `S` のデフォルト値は0です。 Fast Decimal はより高い精度を提供します。
  
  主な最適化点：
  
  ​1. Fast Decimal は可変幅整数を使用して小数を表します。たとえば、有効数字が18以下の小数には64ビット整数を使用します。一方、Decimal V2 はすべての小数に対して一律に128ビット整数を使用します。64ビットプロセッサ上での算術演算や変換演算では、より少ない命令を使用するため、パフォーマンスが大幅に向上します。
  
  ​2. Decimal V2 と比較して、Fast Decimal はいくつかのアルゴリズムで特に乗算において大幅な最適化を行い、パフォーマンスを約4倍向上させています。

Fast Decimal はデフォルトで `enable_decimal_v3` という FE 動的パラメータによって制御されており、デフォルトでは `true` です。

v3.1 以降、StarRocks は [ARRAY](Array.md)、[MAP](Map.md)、および [STRUCT](STRUCT.md) に Fast Decimal エントリをサポートしています。
  
## 制限

StarRocks はHiveのORCおよびParquetファイルからのDECIMALデータのクエリをサポートしていません。

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