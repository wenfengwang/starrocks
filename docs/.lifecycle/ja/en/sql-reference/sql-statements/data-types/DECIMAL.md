---
displayed_sidebar: English
---

# DECIMAL

## 説明

DECIMAL(P[,S])

高精度の固定小数点値。`P`は総有効数字数（精度）を表し、`S`は小数点以下の最大桁数（スケール）を表します。

`P`が省略された場合、デフォルトは10です。`S`が省略された場合、デフォルトは0です。

* Decimal V2

  `P`の範囲は[1,27]、`S`の範囲は[0,9]です。`P`は`S`の値以上でなければなりません。`S`のデフォルト値は0です。

* Fast Decimal（Decimal V3）

  `P`の範囲は[1,38]、`S`の範囲は[0, P]です。`S`のデフォルト値は0です。Fast Decimalはより高い精度を提供します。
  
  主な最適化点：
  
  ​1. Fast Decimalは可変幅整数を使用して小数を表現します。例えば、精度が18以下の小数には64ビット整数を使用します。一方、Decimal V2はすべての小数に対して一律に128ビット整数を使用します。64ビットプロセッサ上での算術演算や変換演算は命令数が少なく、パフォーマンスが大幅に向上します。
  
  ​2. Decimal V2と比較して、Fast Decimalは特に乗算において大幅な最適化を行い、パフォーマンスを約4倍向上させました。

Fast DecimalはFE動的パラメータ`enable_decimal_v3`によって制御され、デフォルトは`true`です。

v3.1以降、StarRocksは[ARRAY](Array.md)、[MAP](Map.md)、[STRUCT](STRUCT.md)内でのFast Decimalの使用をサポートしています。
  
## 制限事項

StarRocksは、HiveからのORCファイルおよびParquetファイル内のDECIMALデータのクエリをサポートしていません。

## 例

テーブル作成時にDECIMALカラムを定義します。

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
