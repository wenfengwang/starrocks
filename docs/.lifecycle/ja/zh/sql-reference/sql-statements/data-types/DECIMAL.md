---
displayed_sidebar: Chinese
---

# DECIMAL

## 説明

DECIMAL(P [, S])

高精度な固定小数点数で、`P` は有効数字の総数（precision）、`S` は小数点以下の最大桁数（scale）を表します。

1.19.0 以降のバージョンでは、DECIMAL 型の（P、S）にデフォルト値が設定されており、デフォルトは Decimal（10、0）です。

例えば：

- 1.19.0 バージョンでは `select cast('12.35' as decimal);` を成功させることができ、P と S を指定する必要はありません。

- 1.19 以前のバージョンでは P と S を明確に指定する必要があります。例：`select cast('12.35' as decimal(5,1));`。指定しない場合、例えば `select cast('12.35' as decimal);` や `select cast('12.35' as decimal(5));` とすると、failed というエラーが表示されます。

### Decimal V2

`P` の範囲は [1,27]、`S` の範囲は [0, 9] です。さらに、`P` は `S` の値以上でなければなりません。デフォルトの `S` の値は 0 です。

### Fast Decimal (1.18 バージョンのデフォルト)

`P` の範囲は [1,38]、`S` の範囲は [0, P] です。デフォルトの `S` の値は 0 です。

StarRocks 1.18 バージョンから、Decimal 型はより高精度の Fast Decimal（別名 Decimal V3）をサポートしています。

主な最適化点は以下の通りです：

1. Decimal の内部表現に複数の幅の整数を使用します。例えば、P ≤ 18 の Decimal 数値には 64-bit 整数を使用します。Decimal V2 が一律に 128-bit 整数を使用するのに対し、64-bit プロセッサ上での算術演算と変換演算により少ない命令を使用するため、パフォーマンスが大幅に向上します。

2. Decimal V2 と比較して、Fast Decimal は特に乗算演算において極限まで最適化されたアルゴリズムを採用しており、パフォーマンスは約 4 倍向上しています。

Fast Decimal 機能は FE の動的パラメータ `enable_decimal_v3` によって制御され、デフォルト値は `true` で、これは機能が有効であることを意味します。

3.1 バージョンからは、[ARRAY](Array.md)、[MAP](Map.md)、[STRUCT](STRUCT.md) が Fast Decimal をサポートしています。

### 使用上の制限

外部 Hive データを読み込む際に、ORC および Parquet ファイル内の Decimal データをサポートしておらず、精度が失われる可能性があります。

## 例

テーブル作成時に DECIMAL 型のフィールドを指定します。

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

decimal, decimalv2, decimalv3, fast_decimal
