---
displayed_sidebar: "Japanese"
---

# covar_samp

## 説明

2つの式のサンプル共分散を返します。この機能はv2.5.10からサポートされています。ウィンドウ関数としても使用できます。

## 構文

```Haskell
COVAR_SAMP(expr1, expr2)
```

## パラメーター

`expr1`と`expr2`はTINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

`expr1`と`expr2`がテーブルの列である場合、この関数はこれらの2つの列のサンプル共分散を計算します。

## 戻り値

DOUBLE 値を返します。式は以下の通りです。ここで、`n`はテーブルの行数を表します:

![covar_samp formula](../../../assets/covar_samp_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n-1}
$$-->

## 使用上の注意

- 2つの列が非NULL値である場合、データ行がカウントされます。それ以外の場合、このデータ行は結果から除外されます。

- `n`が1である場合、0が返されます。

- 入力のいずれかがNULLの場合、NULLが返されます。

## 例

テーブル`agg`に次のデータがあるとします:

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```
`k`と`v`の列のサンプル共分散を計算します:

```plaintext
mysql> select no,COVAR_SAMP(k,v) from agg group by no;
+------+--------------------+
| no   | covar_samp(k, v)   |
+------+--------------------+
|    1 |               NULL |
|    2 | 119.99999999999999 |
+------+--------------------+
```